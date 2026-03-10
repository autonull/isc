# ISC Protocol Specification

> **Purpose**: Detailed P2P networking, DHT, and communication protocol specifications.

---

## Protocol Constants

```typescript
const PROTOCOL_CHAT = '/isc/chat/1.0';
const PROTOCOL_DELEGATE = '/isc/delegate/1.0';
const PROTOCOL_ANNOUNCE = '/isc/announce/1.0';
const PROTOCOL_POST = '/isc/post/1.0';
const PROTOCOL_FOLLOW = '/isc/follow/1.0';
const PROTOCOL_DM = '/isc/dm/1.0';
```

---

## Device Tiers

ISC adapts protocol behavior based on device capability:

| Tier | Target | Model | Relations | ANN | Delegation |
|------|--------|-------|-----------|-----|------------|
| **High** | Desktop/laptop | `all-MiniLM-L6-v2` (22 MB) | All (max 5) | HNSW | Can serve |
| **Mid** | Mid-range phone | `paraphrase-MiniLM-L3-v3` (8 MB) | Root + 2 | HNSW lite | Limited serve |
| **Low** | Budget phone | `gte-tiny` (4 MB) | Root only | Linear scan | Can request |
| **Minimal** | Constrained | Word-hash fallback | Root only | Hamming | Can request |

### Tier Detection

```javascript
async function detectTier() {
  const cores = navigator.hardwareConcurrency ?? 2;
  const mem = navigator.deviceMemory ?? 1;
  const conn = navigator.connection?.effectiveType ?? '4g';

  if (cores >= 4 && mem >= 4 && conn !== '2g') return 'high';
  if (cores >= 2 && mem >= 2) return 'mid';
  if (conn === '2g' || mem < 1) return 'minimal';
  return 'low';
}
```

### Tier-Specific Protocol Parameters

| Tier | numHashes | candidateCap | refreshInterval |
|------|-----------|--------------|-----------------|
| High | 20 | 100 | 5 min |
| Mid | 12 | 50 | 8 min |
| Low | 8 | 20 | 15 min |
| Minimal | 6 | 10 | 20 min |

---

## Libp2p Configuration

### Transports

```javascript
import { webSockets } from '@libp2p/websockets';
import { webRTC } from '@libp2p/webrtc';
import { noise } from '@chainsafe/libp2p-noise';
import { yamux } from '@chainsafe/libp2p-yamux';
import { kadDHT } from '@libp2p/kad-dht';
import { bootstrap } from '@libp2p/bootstrap';

const node = await createLibp2p({
  transports: [webSockets(), webRTC()],
  connectionEncryption: [noise()],
  streamMuxers: [yamux()],
  peerDiscovery: [bootstrap({ 
    list: [
      '/ip4/104.131.131.82/tcp/4001/p2p/QmaCpDMGvV2BGHeYERUEnRQAwe3N8SznbYGzPwpkqDrqEf',
      '/ip4/104.131.131.82/tcp/4001/p2p/QmSoLju6m7xTh3DuokvT5887gRqQofnZ6Gqiq5KhCvv6ip',
    ] 
  })],
  services: { dht: kadDHT({ kBucketSize: 20 }) },
});
await node.start();
```

### Connection Handling

- **Inbound connections**: Accepted from any peer; rate-limited to 20 concurrent
- **Outbound connections**: Dialed on match; max 50 concurrent dials
- **Keepalive**: Heartbeat ping every 30s; drop after 90s silence
- **Reconnect**: On stream drop, wait 5s then attempt one reconnect dial

---

## DHT Protocol

### Channel Schema

```typescript
interface Channel {
  id: string;
  name: string;
  description: string;
  spread: number;           // 0.0-1.0, distribution fuzziness
  relations: Relation[];    // max 5
  createdAt: number;
  updatedAt: number;
}

interface Relation {
  tag: string;              // From ontology below
  object: string;           // Free-form or structured
  weight?: number;          // Default 1.0
}
```

### Relation Ontology

See [SEMANTIC.md](SEMANTIC.md#relation-ontology) for the complete relation ontology with examples and rules.

**Tags:** `in_location`, `during_time`, `with_mood`, `under_domain`, `causes_effect`, `part_of`, `similar_to`, `opposed_to`, `requires`, `boosted_by`

### Key Schema

All DHT keys are prefixed by type for namespace isolation:

```
/isc/announce/<channelID>/<lsh_hash>
/isc/delegate/<peerID>
/isc/mute/<peerID>
/isc/model_registry
/isc/post/<channelID>/<lsh_hash>
/isc/likes/<postID>
/isc/reposts/<postID>
/isc/replies/<postID>
/isc/profile/channels/<peerID>
/isc/follow/<peerID>
/isc/trending/<channelID>
```

### LSH (Locality-Sensitive Hashing)

Vectors are mapped to DHT keys via seeded random-projection LSH:

```javascript
function seededRng(seed: string): () => number {
  let hash = 0;
  for (let i = 0; i < seed.length; i++) {
    const char = seed.charCodeAt(i);
    hash = ((hash << 5) - hash) + char;
    hash = hash & hash;
  }
  let state = Math.abs(hash);
  return () => {
    state = (state * 1103515245 + 12345) & 0x7fffffff;
    return state / 0x7fffffff;
  };
}

function lshHash(vec: number[], channelId: string, numHashes: number = 20, hashLen: number = 32): string[] {
  const rng = seededRng(channelId);
  const hashes: string[] = [];
  for (let i = 0; i < numHashes; i++) {
    const proj = vec.map(() => rng() * 2 - 1);
    const bits = vec.map((v, j) => (v * proj[j] > 0 ? '1' : '0')).join('');
    hashes.push(bits.slice(0, hashLen));
  }
  return hashes;
}
```

**Tier-specific parameters:**

| Tier | numHashes | candidateCap |
|------|-----------|--------------|
| High | 20 | 100 |
| Mid | 12 | 50 |
| Low | 8 | 20 |
| Minimal | 6 | 10 |

### Announcement Payload

```typescript
interface SignedAnnouncement {
  peerID: string;           // libp2p peer ID (base58btc)
  channelID: string;        // channel identifier
  model: string;            // "Xenova/all-MiniLM-L6-v2 @sha256:abc123"
  vec: number[];            // 384-dim embedding vector
  relTag?: string;          // optional relation tag for fused distributions
  ttl: number;              // seconds until expiry (default 300)
  updatedAt: number;        // Unix timestamp (ms)
  signature: Uint8Array;    // ed25519 signature
}
```

### Announcement Loop

```javascript
async function announceChannel(channel: Channel, dists: Distribution[]) {
  const hashes = lshHash(dists[0].mu, channel.id, TIER.numHashes);
  
  for (let i = 0; i < Math.min(hashes.length, dists.length); i++) {
    const payload: SignedAnnouncement = {
      peerID: node.peerId.toString(),
      channelID: channel.id,
      model: LOCAL_MODEL,
      vec: dists[i].mu,
      relTag: dists[i].tag,
      ttl: TIER.refreshInterval * 2,
      updatedAt: Date.now(),
      signature: await sign(encode(payload), keypair.privateKey),
    };
    
    const key = `/isc/announce/${channel.id}/${hashes[i]}`;
    await node.contentRouting.put(key, encode(payload), { ttl: payload.ttl });
  }
}
```

### Query Protocol

```javascript
async function queryProximals(sample: number[], channelId: string): Promise<PeerInfo[]> {
  const seen = new Set<string>();
  const candidates: PeerInfo[] = [];
  const hashes = lshHash(sample, channelId, TIER.numHashes);

  for (const key of hashes) {
    const values = await node.contentRouting.getMany(`/isc/announce/${channelId}/${key}`, {
      count: TIER.candidateCap,
    });
    
    for (const v of values) {
      const peer = decode(v);
      if (peer.peerID === node.peerId.toString()) continue;
      if (peer.model !== LOCAL_MODEL) continue;
      if (!await verify(peer)) continue;
      if (seen.has(peer.peerID)) continue;
      
      seen.add(peer.peerID);
      candidates.push(peer);
    }
  }

  return candidates;
}
```

### TTL & Expiry

- **Default TTL**: 300 seconds (5 minutes)
- **Refresh interval**: Tier-dependent (High: 5 min, Mid: 8 min, Low: 15 min, Minimal: 20 min)
- **Expiry handling**: Clients discard entries where `updatedAt + ttl < Date.now()`
- **Grace period**: Entries within 60s of expiry are still returned but marked stale

---

## Chat Protocol

### Stream Handler

```typescript
interface ChatMessage {
  channelID: string;
  msg: string;
  timestamp: number;
  signature: Uint8Array;
}

async function handleChatStream(stream: Stream) {
  for await (const chunk of stream.source) {
    const msg: ChatMessage = JSON.parse(decode(chunk));
    if (!await verify(msg)) continue;
    displayMessage(msg);
    stream.sink(encode({ ack: msg.timestamp }));
  }
}
```

### Dial Protocol

```javascript
async function initiateChat(peerID: string, channel: Channel): Promise<Stream> {
  const stream = await node.dialProtocol(peerID, PROTOCOL_CHAT);
  
  const greeting: ChatMessage = {
    channelID: channel.id,
    msg: 'Hey, our thoughts are proximal!',
    timestamp: Date.now(),
    signature: await sign(encode(greeting), keypair.privateKey),
  };
  
  await stream.sink(encode(greeting));
  return stream;
}
```

### Group Chat Formation

When 3+ peers match within the same channel:

1. **Centroid computation**: `centroid = mean([vec1, vec2, ...])`
2. **Room ID**: `roomID = sha256(centroid + channelID)`
3. **Announcement**: Each peer announces room membership via DHT
4. **Mesh formation**: Peers dial each other into a floodsub mesh

```typescript
interface GroupRoom {
  roomID: string;
  channelID: string;
  centroid: number[];
  members: string[];  // peerIDs
  createdAt: number;
}
```

---

## Delegation Protocol

See [DELEGATION.md](DELEGATION.md) for the complete specification including request encryption, verification, and trust mechanisms.

**Protocol**: `/isc/delegate/1.0`

---

## Social Layer Protocol

See [SOCIAL.md](SOCIAL.md) for complete social layer protocol specification.

---

## Moderation Protocol

### Mute/Block Events

```typescript
interface MuteEvent {
  type: 'mute';
  muter: string;
  muted: string;
  reason?: string;
  timestamp: number;
  signature: Uint8Array;
}

// Stored in DHT at: /isc/mute/<peerID>
// Clients fetch and cache muted peers locally
```

### Report Events (Phase 2+)

```typescript
interface ReportEvent {
  type: 'report';
  reporter: string;
  reported: string;
  targetPostID?: string;
  reason: 'spam' | 'harassment' | 'impersonation' | 'other';
  description: string;
  timestamp: number;
  signature: Uint8Array;
}
```

---

## Model Registry Protocol

### Registry Entry

```typescript
interface ModelRegistry {
  type: 'model_registry';
  canonical: string;  // "Xenova/all-MiniLM-L6-v2 @sha256:abc123"
  deprecated: string[];
  migrationDeadline: number;  // Unix timestamp
  signature: Uint8Array;  // multisig from maintainers
}
```

### Compatibility Shards

Each model version uses a unique LSH key prefix:

```
v1:abc123:<hash>  // Old model shard
v2:def456:<hash>  // New model shard
```

Clients only query shards matching their loaded model.

---

## Error Handling

### Matching Thresholds

| Range | Label | Protocol Behavior |
|---|---|---|
| 0.85+ | Very Close | Auto-dial enabled |
| 0.70–0.85 | Nearby | Standard candidate |
| 0.55–0.70 | Orbiting | Manual dial required |
| <0.55 | Distant | Filtered out |

### Stream Errors

```typescript
enum StreamError {
  TIMEOUT = 'TIMEOUT',
  INVALID_SIGNATURE = 'INVALID_SIGNATURE',
  MODEL_MISMATCH = 'MODEL_MISMATCH',
  RATE_LIMITED = 'RATE_LIMITED',
  NAT_UNREACHABLE = 'NAT_UNREACHABLE',
}

async function handleStreamError(err: StreamError, peerID: string) {
  switch (err) {
    case StreamError.INVALID_SIGNATURE:
    case StreamError.MODEL_MISMATCH:
      await blockPeer(peerID);
      break;
    case StreamError.RATE_LIMITED:
      await backoff(60000);
      break;
    case StreamError.NAT_UNREACHABLE:
      await tryCircuitRelay(peerID);
      break;
  }
}
```

### DHT Errors

| Error | Handling |
|---|---|
| `KEY_NOT_FOUND` | Retry with exponential backoff (max 3 attempts) |
| `QUOTA_EXCEEDED` | Reduce announcement frequency; notify user |
| `CONNECTION_CLOSED` | Reconnect to bootstrap; resume announcements |
| `INVALID_VALUE` | Discard; log for analytics |

---

## Rate Limits

| Operation | Scope | Limit | Enforcement |
|---|---|---|---|
| DHT Announce | per peer / min | 5 | Client + supernode |
| Delegation Request | per peer / min | 3 | Supernode |
| Delegation Response | per supernode concurrent | 10 | Supernode |
| Chat Dial | per peer / hr | 20 | Client |
| DHT Query | per peer / min | 30 | Bootstrap relay |

---

## Message Encoding

All messages use **CBOR** for compact binary encoding:

```javascript
import * as cbor from 'cbor-x';

function encode(obj: any): Uint8Array {
  return cbor.encode(obj);
}

function decode(data: Uint8Array): any {
  return cbor.decode(data);
}
```

Fallback: JSON for debugging.

---

## Bootstrap Peers

Default public libp2p relays:

```
/ip4/104.131.131.82/tcp/4001/p2p/QmaCpDMGvV2BGHeYERUEnRQAwe3N8SznbYGzPwpkqDrqEf
/ip4/104.131.131.82/tcp/4001/p2p/QmSoLju6m7xTh3DuokvT5887gRqQofnZ6Gqiq5KhCvv6ip
```

---

## Testing

### Local Supernode Testing

```bash
# Terminal 1: Supernode
npx serve . --port 8080
# Open http://localhost:8080?tier=high&supernode=true

# Terminal 2: Low-tier client
npx serve . --port 8081
# Open http://localhost:8081?tier=low&delegate=true
```

Verify in DevTools:
- Console: `peer.delegation.stats`
- Network: Encrypted messages on `/isc/delegate/1.0`
- Match forms within 30 seconds

### Debug Logging

```
Settings → Developer → Enable Debug Logging
```

Outputs: DHT announcements, match queries, delegation, WebRTC events.

---

## Version Negotiation

Protocol versions are negotiated via libp2p multiselect:

```
/isc/chat/1.0
/isc/delegate/1.0
/isc/announce/1.0
/isc/post/1.0
```

Future versions: `/isc/chat/2.0`, etc. Clients support backward compatibility within major versions.
