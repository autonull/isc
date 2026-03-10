# ISC Security Specification

> **Purpose**: Detailed threat model, safety mechanisms, privacy guarantees, and authenticity protocols.
>
> For an overview, see [README.md](README.md#security--safety).

---

## Overview

ISC achieves strong authenticity, safety, and privacy properties by building on its fully browser-based, P2P architecture. It starts with structural advantages over centralized platforms:

- No central server collecting data
- No persistent profiles beyond user-controlled local storage
- Ephemeral announcements (TTLs prevent long-term tracking)
- E2E encryption for all communications

The design is inspired by **Nostr** (cryptographic authenticity, censorship resistance, relay resilience), adopting its best elements while extending them with ISC's vector-space primitives.

---

## Authenticity

### Built-in (Libp2p Baseline)

- **Libp2p peer IDs**: Cryptographic identity via Noise protocol + ed25519 keys
- **Authenticated announcements**: All DHT announcements signed with private key
- **Encrypted streams**: WebRTC DTLS secures all chat communications

### User Keypairs

On first launch:

```javascript
import { generateKeypair } from '@isc/core';

// Generate ed25519 keypair via Web Crypto API
const keypair = await generateKeypair();

// Public key = user's persistent "ISC identity" (like Nostr's npub)
const userID = await toBase58(keypair.publicKey);

// Private key stored in IndexedDB, optionally encrypted with passphrase
await storage.set('keypair', {
  publicKey: keypair.publicKey,
  privateKey: await encryptPrivateKey(keypair.privateKey, passphrase),
});
```

### Signed Announcements

Every DHT announcement is signed:

```typescript
interface SignedAnnouncement {
  peerID: string;
  channelID: string;
  model: string;
  vec: number[];
  relTag?: string;
  ttl: number;
  updatedAt: number;
  signature: Uint8Array;  // ed25519 signature
}

// Verification
async function verifyAnnouncement(ann: SignedAnnouncement): Promise<boolean> {
  const payload = encode({
    peerID: ann.peerID,
    channelID: ann.channelID,
    model: ann.model,
    vec: ann.vec,
    relTag: ann.relTag,
    ttl: ann.ttl,
    updatedAt: ann.updatedAt,
  });
  return verify(payload, ann.signature, getPublicKey(ann.peerID));
}
```

### Signed Posts

All social-layer content carries signatures:

```typescript
interface SignedPost {
  type: 'post';
  postID: string;
  author: string;
  content: string;
  timestamp: number;
  signature: Uint8Array;
}

// Tamperproof by design; no central authority required
```

---

## Safety

### Structural Baseline

| Property | Benefit |
|---|---|
| **Ephemeral TTLs** | Announcements expire; no long-term tracking |
| **No persistent profiles** | Nothing to scrape or dox |
| **WebRTC DTLS** | E2E encryption for all chat streams |

### Deployment Modes

ISC supports two deployment modes with different safety assumptions. See [README.md](README.md#deployment-modes) for the complete specification.

**Trusted Network (Phase 1):** Pre-existing social trust, rate limiting + mute/block + semantic filters

**Public Network (Phase 2+):** Open participation, reputation weighting + stake signaling + coherence checks + decentralized moderation

---

## Implemented Mechanisms (Trusted Network Mode)

### Rate Limiting

Rate limits prevent spam and abuse. See [PROTOCOL.md](PROTOCOL.md#rate-limits) for the complete specification.

**Key limits:** DHT Announce: 5/min, Chat Dial: 20/hr, DHT Query: 30/min

### Mute / Block Lists

```typescript
interface MuteEvent {
  type: 'mute';
  muter: string;      // peerID
  muted: string;      // peerID
  reason?: string;
  timestamp: number;
  signature: Uint8Array;
}

// Stored in DHT at: /isc/mute/<peerID>
// Clients fetch and cache muted peers locally
// Auto-filter flagged peers from match results
```

### Semantic Filters

- **Minimum similarity threshold**: Default 0.55 cosine similarity
- **Per-channel controls**: Users define their own "safe zone"
- **Announcements below threshold**: Not returned to users

### Harassment Exit

- **Auto-decay**: Chats exit naturally when similarity drops below threshold (thought drift)
- **One-click mute**: With propagation to connected peers
- **No forced interactions**: Users always control who can contact them

---

## Planned Mechanisms (Public Network Mode — Phase 2)

### Reputation Weighting

```typescript
interface ReputationScore {
  peerID: string;
  score: number;       // 0.0 - 1.0
  interactions: number;
  halfLifeDays: number; // 30-day decay
  lastUpdated: number;
}

// Peers accumulate reputation via successful interactions
// Low-rep announcements deprioritized in ANN results
// Reputation decays with 30-day half-life
```

**Sybil resistance**:
- Mutual signing requirement (both parties confirm interaction)
- Time-weighted decay
- 7-day bootstrapping period for new peers

### Stake-Based Signaling (Opt-In)

```typescript
interface StakeProof {
  peerID: string;
  amount: number;      // Satoshis locked
  lockTx: string;      // Lightning transaction ID
  expiry: number;
  signature: Uint8Array;
}

// Users may lock Lightning satoshis as sybil-resistance signal
// Slashed on verified abuse
// Never required for basic use
```

### Semantic Coherence Checks

```javascript
function checkCoherence(announcement: SignedAnnouncement): boolean {
  const descriptionEmbed = await embed(announcement.channelDescription);
  const vecDistance = cosineSimilarity(descriptionEmbed, announcement.vec);
  
  // Announcements with embeddings far from stated description are flagged
  return vecDistance > 0.4;  // Threshold configurable
}

// Flagged announcements reviewed by high-rep supernodes
```

### Decentralized Moderation

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

// Signed reports stored in DHT
// Clients weight reports by reporter reputation
// No central moderation team; safety emerges from network geometry
```

---

## Privacy

### Built-in Baseline

| Property | Implementation |
|---|---|
| **No central servers** | All data lives locally or traverses P2P |
| **Vector-only announcements** | Only vectors + peerID announced publicly |
| **Raw text never broadcast** | Unless user explicitly posts |
| **E2E encrypted chats** | WebRTC DTLS + Noise protocol |

### Enhanced Capabilities

#### Ephemeral Keys

```javascript
// Optional throwaway keypairs per session or channel
async function createEphemeralIdentity(): Promise<Keypair> {
  const keypair = await generateKeypair();
  // No persistent npub linkage
  return keypair;
}
```

#### IP Protection

- **Libp2p circuit relays**: Obfuscate direct IPs
- **Public STUN/TURN fallbacks**: NAT traversal without IP leakage
- **Optional Tor / I2P routing**: Community libp2p transport plugins for high-privacy users

#### Metadata Minimization

- **Vector-only announcements**: Raw text revealed only in direct WebRTC chats
- **No cross-session tracking**: Without explicit follows
- **Plausible deniability**: Channel spread (σ) adds deliberate fuzz to announced position

#### Data Sovereignty

```javascript
// Users control full export/delete of all local data
async function exportUserData(): Promise<UserDataExport> {
  return {
    keypair: await storage.get('keypair'),
    channels: await storage.get('channels'),
    history: await storage.get('chat_history'),
    follows: await storage.get('follows'),
  };
}

async function deleteAllData(): Promise<void> {
  await indexedDB.deleteDatabase('isc');
  localStorage.clear();
}
```

#### Delegation Privacy

| Guarantee | Implementation |
|---|---|
| **Request encryption** | Encrypted with supernode's public key |
| **Minimal exposure** | Only channel descriptions delegated (never chat messages) |
| **Per-channel control** | Users can disable delegation via Settings |
| **No logging policy** | Supernodes expected to discard request contents after computation |
| **Future: ZK proofs** | Verification without revealing inputs |

---

## Threat Model

ISC operates under the following security assumptions:

| Threat | Assumption | Mitigation (Phase 1: Trusted) | Mitigation (Phase 2+: Public) |
|--------|------------|-------------------------------|-------------------------------|
| **Malicious supernodes** | Honest-but-curious; may log requests or return incorrect embeddings | Local sanity checks + trusted operator selection | + Reputation weighting + optional SNARK proofs (future) |
| **DHT bootstrap peers** | Not actively adversarial; may go offline | Multiple bootstrap peers; graceful reconnect | Same |
| **Sybil attackers** | Can create many identities | Social trust barrier (invite-only) | Reputation decay + uptime history + opt-in stake |
| **Network eavesdroppers** | Can observe traffic patterns but not decrypt content | WebRTC DTLS + Noise protocol + E2E encryption | Same |
| **Browser compromise** | XSS vulnerabilities possible; IndexedDB accessible | Passphrase encryption for private keys (optional) | Same + hardware wallet integration (future) |
| **Model poisoning** | Attacker distributes malicious embedding model | Canonical model registry (DHT-hosted, signed) | Same |
| **Reputation gaming** | Attacker creates fake positive interactions | N/A (reputation not yet enabled) | Mutual signing + time-weighted decay |

---

## Explicitly Out of Scope

| Threat | Rationale |
|---|---|
| **Browser zero-day exploits** | Mitigated by regular dependency updates |
| **User key compromise without passphrase** | Mitigated by optional passphrase encryption |
| **Physical device theft** | User responsibility; future: hardware key support |
| **Government-level traffic correlation** | Tor/I2P integration mitigates but doesn't eliminate |

---

## Security Review Checklist

Before merging delegation-related or cryptographic PRs, ensure:

- [ ] All delegated responses are cryptographically signed (ed25519)
- [ ] Local verification logic has unit tests for edge cases (invalid sigs, malformed embeddings, timeout)
- [ ] Rate limiting is enforced on both request and response paths
- [ ] Fallback behavior is tested (no supernodes available, network partition)
- [ ] Memory usage is bounded (no unbounded request queues; max 100 pending delegations)
- [ ] Encryption keys are never logged or exposed in DevTools
- [ ] Model version checks prevent cross-model similarity computation
- [ ] Reputation system (if applicable) resists Sybil attacks (mutual signing, decay)
- [ ] DHT announcements include TTL and are signed
- [ ] WebRTC streams use DTLS encryption (verify via browser DevTools)
- [ ] Accessibility audit (NVDA, VoiceOver) completed
- [ ] Browser compromise mitigations: XSS testing completed; passphrase encryption recommended for high-risk users

**PRs failing any checklist item will be rejected without review.**

---

## Comparison with Other Platforms

| Aspect | ISC | Nostr | X (Centralized) |
|---|---|---|---|
| **Authenticity** | Keypair signing + libp2p | Keypair signing | Platform verification |
| **Safety** | Layered anti-spam + semantic filters + mutes | Client-side + propagation | Central bans + biases |
| **Privacy** | No servers; optional Tor; ephemeral keys | Public-by-default; Tor mitigable | Full surveillance |
| **Censorship resistance** | DHT + WebRTC; no deplatforming | Relay-based; resilient | Platform-controlled |

---

## Key Management Best Practices

### For Users

1. **Enable passphrase encryption** in Settings (high-risk users)
2. **Backup your keypair** via Settings → Export
3. **Use ephemeral keys** for sensitive channels
4. **Rotate keys periodically** if concerned about compromise

### For Developers

1. **Never log private keys** or raw encryption material
2. **Use Web Crypto API** for all cryptographic operations
3. **Clear sensitive data** from memory after use
4. **Implement secure key derivation** for passphrase encryption
5. **Test XSS vectors** thoroughly; IndexedDB is accessible to malicious scripts

---

## Incident Response

### Key Compromise

1. User generates new keypair
2. User announces key rotation via signed message from old key (if possible)
3. User updates follows/mutes under new identity
4. Old key is marked as compromised in local cache

### Supernode Misbehavior

1. Peer detects invalid/malicious response
2. Peer blocks supernode locally
3. Peer broadcasts signed report to DHT
4. High-rep peers review; if confirmed, supernode is deprioritized network-wide

### Model Poisoning Attempt

1. Client detects unknown model hash in announcement
2. Client discards announcement; logs for analytics
3. If widespread, community proposes model registry update
4. Clients auto-update on next launch
