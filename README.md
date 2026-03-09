# ISC — Internet Semantic Chat

> *Meet your thought neighbors.*

A fully **decentralized, browser-only** social platform that uses lightweight in-browser LLM embeddings to place your ideas in a shared vector space — then serendipitously connects you with peers whose mental distributions are closest to yours. No servers. No accounts. No surveillance. Just thought and conversation.

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](#license)
[![Browser-only](https://img.shields.io/badge/runs%20in-browser-brightgreen)](#tech-stack)
[![P2P](https://img.shields.io/badge/network-libp2p%20%2B%20WebRTC-orange)](#tech-stack)

---

## Table of Contents

- [Overview](#overview)
- [Deployment Modes](#deployment-modes)
- [Channels](#channels)
  - [Channel Schema](#channel-schema)
  - [Matching Continuity](#matching-continuity)
- [Semantic Model](#semantic-model)
  - [Relation Ontology](#relation-ontology)
  - [Compositional Embeddings](#compositional-embeddings)
  - [Relational Matching](#relational-matching)
- [Embedding Standards](#embedding-standards)
  - [Model Version Negotiation](#model-version-negotiation)
- [Architecture](#architecture)
  - [1. Peer Initialization](#1-peer-initialization)
  - [2. Channels & Distributions](#2-channels--distributions)
  - [3. Announcing to the DHT](#3-announcing-to-the-dht)
  - [4. Querying for Proximals](#4-querying-for-proximals)
  - [5. Forming Chats](#5-forming-chats)
  - [6. Handling Dynamics](#6-handling-dynamics)
  - [7. Supernode Delegation Architecture](#7-supernode-delegation-architecture)
  - [8. Threat Model](#8-threat-model)
- [Device Tiers](#device-tiers)
- [Tech Stack](#tech-stack)
- [Safety, Privacy & Authenticity](#safety-privacy--authenticity)
  - [Authenticity](#authenticity)
  - [Safety](#safety)
  - [Privacy](#privacy)
- [Social Network Layer](#social-network-layer)
  - [Posts & Feeds](#posts--feeds)
  - [Interactions](#interactions)
  - [Profiles & Follows](#profiles--follows)
  - [Communities](#communities)
  - [Exceeding Centralized Platforms](#exceeding-centralized-platforms)
- [Getting Started](#getting-started)
  - [Testing Supernode Delegation Locally](#testing-supernode-delegation-locally)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
  - [Security Review Checklist](#security-review-checklist)
- [License](#license)

---

## Overview

ISC is an experiment in **semantic proximity networking**. Instead of joining a chat room by topic tag or URL, you describe your current thoughts — and the system finds other people thinking similar things right now.

Key properties:

| Property | Detail |
|---|---|
| **Fully browser-native** | Zero server-side compute; all embedding and routing runs in-browser |
| **Serverless P2P** | Kademlia DHT for discovery, WebRTC DataChannels for chat |
| **Channels** | Maintain multiple named presence contexts with optional relational semantics |
| **Adaptive by device** | Model size, worker concurrency, and matching depth auto-scale to device capability |
| **Fuzzy identity** | You're represented as a *distribution* in embedding space, not a fixed point |
| **Relational hypergraph** | Optional relation tags compose subjects with context (location, time, mood, causality) into unified embedding manifolds |
| **Ephemeral by default** | Announcements expire (TTL-based); no persistent profile |
| **E2E encrypted** | WebRTC DTLS + keypair signing secures all streams |
| **Social-ready** | Posts, feeds, follows, communities — all geometry-native |

The key insight: rather than routing by topic *labels*, ISC routes by *semantic geometry*. Two people thinking about "the ethics of AI art" and "copyright in machine creativity" will find each other even if they used completely different words.

---

## Deployment Modes

ISC is designed for **progressive decentralization**, starting with trusted networks and evolving toward open public participation:

| Mode | Phase | Use Case | Trust Assumptions | Safety Features |
|------|-------|----------|-------------------|-----------------|
| **Trusted Network** | Phase 1 | Private communities, organizations, invite-only groups | Pre-existing social trust; members known or vouched for | Rate limiting, mute/block, semantic filters |
| **Federated Networks** | Phase 2 | Interconnected trusted communities; reputation bridges | Trust within communities; verified cross-community links | Reputation weighting, signed moderation events |
| **Public Network** | Phase 2+ | Open participation; anyone can join | Potential adversaries; sybil attacks possible | Stake signaling, coherence checks, decentralised moderation |

**Design rationale**: Building for trusted networks first allows ISC to:
- Launch with minimal safety infrastructure (no complex reputation system required)
- Validate core semantic matching and delegation in low-adversary environments
- Iterate on user experience before hardening against sophisticated attacks
- Grow organically through community invitations rather than cold-start problems

---

## Channels

A **channel** is a named, editable presence context — your stated cluster of thoughts or interests at a given moment. You can maintain several channels simultaneously and switch between them; each one independently announces your position in semantic space and accumulates its own match history and chat sessions.

```
┌─────────────────────────────────────────────────┐
│  Channels                          [+ New]       │
│ ─────────────────────────────────────────────── │
│  ● Work         "distributed systems, consensus" │
│  ○ Evening      "ambient music, slow fiction"    │
│  ○ Weekend      "ceramics, fermentation"         │
└─────────────────────────────────────────────────┘
```

**How channels work:**

- Each channel has a **name**, a **description** (free-form text, the seed of your embedding), an optional **spread** σ (how wide your distribution is in the space), and optional **relations** (contextual bindings — see [Semantic Model](#semantic-model)).
- Channels are **stored locally** in `localStorage` / IndexedDB — they persist across sessions and survive browser refresh.
- When a channel is **active**, its distribution is announced to the DHT and refreshed on a timer. Inactive channels are silently withdrawn (TTL expiry).
- You can **edit** a channel's description or relations at any time; the embedding recomputes and the DHT announcement updates within seconds.
- Channels are **independent**: switching to "Evening" does not collapse your "Work" chats — in-progress conversations stay live until explicitly closed or timed out.
- Channels can be **reordered**, **duplicated** (to fork an idea), or **archived** (withdrawn from DHT but kept locally).
- Each channel maintains its own **match list** and **chat history** (session-local; optionally persisted).

### Channel Schema

```json
{
  "id": "ch_ai_ethics_9b2f",
  "name": "AI Ethics",
  "description": "Ethical implications of machine learning and autonomy",
  "spread": 0.15,
  "relations": [
    {
      "tag": "in_location",
      "object": "lat:35.6895, long:139.6917, radius:50km",
      "weight": 1.2
    },
    {
      "tag": "during_time",
      "object": "start:2026-01-01T00:00:00Z, end:2026-12-31T23:59:59Z"
    },
    {
      "tag": "with_mood",
      "object": "reflective and cautious"
    }
  ],
  "createdAt": 1741400000,
  "updatedAt": 1741450000
}
```

**Rules**: Max 5 relations (UI-enforced). Tags must come from the [Relation Ontology](#relation-ontology). Objects are free-form text or structured strings for spatiotemporal relations.

**Implementation phases:**
- **Phase 1 (Trusted Network)**: Basic multi-channel support — create, switch, and manage multiple channels; single-channel semantic matching; 1:1 chats.
- **Phase 2 (Federated Networks)**: Relational embeddings — cross-channel semantic composition; multi-channel match aggregation; channel groupings.

### Matching Continuity

ISC uses **approximate, ranked matching** — not discrete rooms. Nothing forces you into a fixed bucket:

| Aspect | How ISC Handles It | Degree of Continuity |
|---|---|---|
| Single vs. group chat | Starts 1:1; groups auto-form on density | High — can stay 1:1 forever |
| Match discovery | Ranked list from ANN re-rank | Very high — full gradient of proximity |
| Joining conversations | Direct dial or centroid-based room join | Medium-high — rooms are opt-in |
| Seeing loose neighbors | Yes, via ranked candidates (top-k beyond groups) | High — explicit support for approximates |
| Dynamic repositioning | Edit channel → instant re-query/announce | Very high — thoughts aren't locked |
| Cross-topic serendipity | Chaos mode + spread slider + distribution matching | High — encourages fuzzy orbits |

Thresholds are configurable (default ~0.7 similarity), and the UI can display a sliding scale: **Very Close (0.85+)**, **Nearby (0.70–0.85)**, **Orbiting (0.55–0.70)**. The ranked match list is the primary UI — groups are a pragmatic convenience for dense swarms, not a hard silo.

---

## Semantic Model

ISC's semantic model achieves **full relational semantic expressivity** within browser constraints. The core shift: channels are not just distributions in a flat vector space — they form a **relational hypergraph** where optional relation tags bind concepts compositionally into unified embedding manifolds.

"AI ethics in Neo-Tokyo during 2026" is not a filter applied post-hoc; it is **represented as a single discoverable embedding** that differs geometrically from "Neo-Tokyo AI ethics" — the model captures directional binding.

### Relation Ontology

A fixed set of 10 predefined relation tags ensures network-wide interoperability:

| Tag | Meaning |
|---|---|
| `in_location` | Spatiotemporal place (lat/long/radius or named place) |
| `during_time` | Temporal window (ISO 8601 start/end) |
| `with_mood` | Emotional or tonal context |
| `under_domain` | Categorical scope or discipline |
| `causes_effect` | Causal link between subject and outcome |
| `part_of` | Compositional or hierarchical membership |
| `similar_to` | Analogical tie |
| `opposed_to` | Contrastive relation |
| `requires` | Dependency or prerequisite |
| `boosted_by` | Amplification or modulation |

The ontology is fixed for stability; community forks may extend it in future major versions.

### Compositional Embeddings

For each channel, a set of distributions is computed locally:

1. **Root**: `Embed(description)` → μ\_root, σ = channel.spread
2. **Per-relation fused distributions**: For each relation, compose the prompt `"${description} ${tag} ${object}"` and embed → μ\_fused, σ\_fused

```javascript
async function computeRelationalDistributions(channel) {
  const extractor = await pipeline('feature-extraction', MODEL_FOR_TIER);
  const rootEmb = await extractor(channel.description, { pooling: 'mean', normalize: true });
  const dists = [{ type: 'root', mu: Array.from(rootEmb.data), sigma: channel.spread }];

  for (const rel of channel.relations ?? []) {
    let obj = rel.object;
    if (['in_location', 'during_time'].includes(rel.tag)) {
      const parsed = parseSpatiotemporal(rel.tag, obj);
      obj = `${obj} (${JSON.stringify(parsed)})`;
    }
    const prompt = `${channel.description} ${rel.tag.replace('_', ' ')} ${obj}`;
    const emb = await extractor(prompt, { pooling: 'mean', normalize: true });
    dists.push({
      type: 'fused',
      tag: rel.tag,
      mu: Array.from(emb.data),
      sigma: channel.spread / (rel.weight ?? 1.0),
      weight: rel.weight ?? 1.0,
    });
  }
  return dists;
}
```

**Tier fallback**: Low tier announces root distribution only. Mid tier announces root + up to 2 key relations. High tier announces all.

### Relational Matching

Matching uses bipartite alignment across the multi-vector hypergraph:

```javascript
function relationalMatch(myDists, peerDists) {
  let score = 0, totalWeight = 0;

  // Root alignment
  score += cosineSimilarity(myDists[0].mu, peerDists[0].mu);
  totalWeight += 1;

  // Fused alignments — best-match bipartite pairing
  for (let i = 1; i < myDists.length; i++) {
    let best = 0;
    for (let j = 1; j < peerDists.length; j++) {
      const sim = cosineSimilarity(myDists[i].mu, peerDists[j].mu)
                  * (myDists[i].tag === peerDists[j].tag ? 1.2 : 1.0);
      best = Math.max(best, sim);
    }
    // Spatiotemporal domain boost
    if (['in_location', 'during_time'].includes(myDists[i].tag)) {
      best += spatiotemporalSimilarity(myDists[i].tag, myDists[i], peerDists) * 0.5;
    }
    score += best * myDists[i].weight;
    totalWeight += myDists[i].weight;
  }

  return score / totalWeight; // > 0.75 = match
}
```

Monte Carlo sampling runs over `TIER.mcSamples` draws from each distribution before scoring (High/Mid tier only).

---

## Embedding Standards

> **Mixing models breaks the network.** All active peers must use vectors from the **same model family** for cosine similarity to be meaningful across DHT candidates.

Embedding spaces from different models are not directly comparable, even at identical dimensionality. Cross-model cosine scores are often arbitrary or near-random — matching degrades to noise.

**Canonical model ladder** (all available via `@xenova/transformers.js` / ONNX-WASM, no server):

| Model (Xenova ID) | Dims | Size | Tier | Notes |
|---|---|---|---|---|
| `all-MiniLM-L6-v2` | 384 | ~22 MB | **High (default)** | Best balance of speed, quality, and cross-peer consistency |
| `bge-small-en-v1.5` | 384 | ~18 MB | High (alt) | Top retrieval benchmarks; strong alternative to MiniLM |
| `paraphrase-MiniLM-L3-v3` | 384 | ~8 MB | **Mid** | Lighter, still aligned with MiniLM family |
| `gte-tiny` | 384 | ~4 MB | **Low** | Minimal compute; same-family fallback |
| Word-hash fallback | — | 0 MB | **Minimal** | No model; Hamming distance on predefined word hashes |

**Rules enforced in code:**

- One model loaded per tier; never allow arbitrary user-selected models in production.
- Every DHT announcement includes a `model` field (e.g., `"Xenova/all-MiniLM-L6-v2"`).
- During query refinement, discard candidates whose `model` field does not match the local model. The network self-partitions by model version without fully breaking.

```json
// DHT announcement payload (extended)
{
  "peerID":    "12D3KooW...",
  "channelID": "ch_ai_ethics_9b2f",
  "model":     "Xenova/all-MiniLM-L6-v2",
  "vec":       [0.12, -0.07, ...],
  "relTag":    "in_location",
  "ttl":       300
}
```

### Model Version Negotiation

To prevent network fragmentation across embedding model versions:

1. **Announcement field**: Every DHT entry includes `model_version` with hash:
   ```json
   "model": "Xenova/all-MiniLM-L6-v2 @sha256:abc123def456"
   ```

2. **Compatibility groups**: Clients maintain a `supported_models` list. During query refinement, candidates with unsupported models are silently filtered. Peers announce their supported models in bootstrap handshake.

3. **Graceful migration**: When a new canonical model is adopted:
   - Old-model peers continue operating in a "compatibility shard" (separate LSH buckets)
   - Dual-announcement mode (optional, High-tier only) allows peers to announce in both old and new spaces for 90 days
   - After 90 days, old-model announcements expire naturally via TTL
   - Clients show migration prompt when >50% of matches use newer model

4. **Community model registry**: A signed, DHT-hosted manifest lists approved model versions and migration timelines:
   ```json
   {
     "type": "model_registry",
     "canonical": "Xenova/all-MiniLM-L6-v2 @sha256:abc123",
     "deprecated": ["Xenova/all-MiniLM-L6-v2 @sha256:old456"],
     "migrationDeadline": 1772936000,
     "signature": "community_multisig_here"
   }
   ```

5. **Never mix models**: Cosine similarity is only meaningful within the same embedding space. Clients must never compute similarity across different model versions.

> **Model update process**: Community proposes new model → 30-day comment period → Multisig signing by trusted maintainers → DHT registry update → Clients auto-fetch on next launch.

---

## Architecture

### 1. Peer Initialization

When a user loads the static app (hostable on IPFS or GitHub Pages), the client:

1. Runs the capability probe → selects a device tier
2. Generates or restores a **keypair** (ed25519 via Web Crypto API) for signing
3. Loads and caches the appropriate embedding model
4. Restores saved channels from `localStorage`
5. Spins up the libp2p node

```javascript
import { createLibp2p }  from 'libp2p';
import { webSockets }    from '@libp2p/websockets';
import { webRTC }        from '@libp2p/webrtc';
import { noise }         from '@chainsafe/libp2p-noise';
import { yamux }         from '@chainsafe/libp2p-yamux';
import { kadDHT }        from '@libp2p/kad-dht';
import { bootstrap }     from '@libp2p/bootstrap';

const node = await createLibp2p({
  transports:           [webSockets(), webRTC()],
  connectionEncryption: [noise()],
  streamMuxers:         [yamux()],
  peerDiscovery:        [bootstrap({ list: ['/ip4/.../tcp/.../p2p/Qm...'] })],
  services: { dht: kadDHT({ kBucketSize: 20 }) },
});
await node.start();
```

Bootstrap peers can be any well-known libp2p relay (public ones are listed in the libp2p docs, or run your own seed tab).

---

### 2. Channels & Distributions

When a channel is activated, its description (and any relations) are embedded and distributions constructed via [`computeRelationalDistributions`](#compositional-embeddings).

Matching strategies (selectable per channel):
- **Point match** — single sampled point from root distribution (fast; good for Low/Min tier)
- **Distribution match** — approximate EMD / KL via Monte Carlo samples (richer; High/Mid tier)
- **Relational match** — full hypergraph alignment across all fused distributions (High tier)

---

### 3. Announcing to the DHT

Vectors are mapped to DHT keys via **seeded Locality-Sensitive Hashing (LSH)**. The seed is derived from the channel ID for stable, reproducible bucket placement across reconnects.

```javascript
function lshHash(vec, channelId, numHashes, hashLen = 32) {
  const rng = seededRng(channelId);
  return Array.from({ length: numHashes }, () => {
    const proj = vec.map(() => rng() * 2 - 1);
    return vec.map((v, j) => (v * proj[j] > 0 ? '1' : '0')).join('').slice(0, hashLen);
  });
}
```

For relational channels, each fused distribution is announced separately with a prefixed key (e.g., `"in_location:<hash>"`), enabling relation-specific discovery. Entries per channel are capped (max 5 across root + relations).

All payloads are signed with the peer's private key before `contentRouting.put`; recipients verify signatures and discard unsigned or mismatched-model candidates.

- The active channel's announce loop runs on `TIER.refreshInterval`.
- On connectivity loss, the loop pauses and resumes automatically on reconnect.

---

### 4. Querying for Proximals

```javascript
const seen = new Set();
const candidates = [];

for (const key of lshHash(currentSample, channel.id, TIER.numHashes)) {
  const values = await node.contentRouting.getMany(key, { count: TIER.candidateCap });
  for (const v of values) {
    const peer = JSON.parse(v);
    if (peer.peerID === node.peerId.toString()) continue; // skip self
    if (peer.model !== LOCAL_MODEL) continue;             // skip mismatched models
    if (!verifySignature(peer)) continue;                 // skip invalid signatures
    if (seen.has(peer.peerID)) continue;
    seen.add(peer.peerID);
    candidates.push(peer);
  }
}
```

Candidates are refined locally using usearch (HNSW) by relational similarity score, filtering to top-k (default 10–20) above threshold 0.75. On Low/Minimal tier, a linear scan over the smaller candidate set replaces HNSW.

The ranked list auto-refreshes in the background every 30–60 s so new peers drifting into proximity appear without manual action.

---

### 5. Forming Chats

Top matches are dialed directly over **WebRTC** via a custom libp2p protocol:

```javascript
const stream = await node.dialProtocol(peerID, '/isc/chat/1.0');
stream.write(JSON.stringify({
  channelID: channel.id,
  msg: 'Hey, our thoughts are proximal!',
}));
```

**Group chats**: when 3+ peers match within the same channel, each dials the others into a mesh. Libp2p floodsub handles broadcast. Rooms are identified by hashing the group's shared vector centroid — announced via DHT so latecomers can join.

**Reconnect logic**: on stream drop, ISC waits 5 s then attempts one reconnect dial. On failure, the peer is marked as lost, the UI notifies the group, and a fresh query finds replacements.

Group formation is fully optional — a UI toggle ("Prefer 1:1 only" / "Auto-mesh dense clusters") lets users stay in ranked 1:1 connections if they prefer.

---

### 6. Handling Dynamics

| Challenge | Approach |
|---|---|
| **Peer churn** | DHT naturally handles leaves; heartbeat pings every 30 s drop dead partners |
| **Cold start** | Bootstrap via public libp2p relays; "seed tab" pattern for early adopters |
| **Scale (1k+ peers)** | LSH + DHT approximates well; gossip (peers share neighbour lists) for > 10k |
| **Thought drift** | Edit a channel description → new embedding pushed to DHT within seconds |
| **NAT traversal** | Public STUN/TURN servers as fallback; libp2p circuit relays for hard NATs |
| **Offline / flaky net** | Announce loop pauses on `offline` event; resumes on `online`; in-flight chats queue messages |
| **Stale candidates** | TTL on all DHT entries; `updatedAt` timestamp lets client discard old results |
| **Duplicate peers** | Client-side dedup by `peerID` before ANN ranking |
| **Model fragmentation** | `model` field in payload; clients silently discard mismatched-model candidates |
| **Channel collision** | Channel IDs are random; per-channel LSH seeds prevent cross-channel bucket contamination |
| **Spam** | Phase 1: rate limits | Phase 2+: + reputation, stake, coherence checks |

---

### 7. Supernode Delegation Architecture

ISC supports **capability-aware delegation**: high-tier peers optionally act as *supernodes* to assist Low/Minimal-tier peers with computationally expensive operations—without compromising decentralization or privacy.

#### Delegation Flow

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Low-Tier Peer  │────▶│   Supernode     │────▶│  DHT / Network  │
│  (Mobile)       │     │  (Desktop)      │     │                 │
└─────────────────┘     └─────────────────┘     └─────────────────┘
        │                       │                       │
        │ 1. Request assistance │                       │
        │    - Embed text       │                       │
        │    - Run ANN query    │                       │
        │    - Verify sigs      │                       │
        │◀──────────────────────│                       │
        │ 2. Receive results    │                       │
        │    - Encrypted        │                       │
        │    - Signed           │                       │
        │    - Locally verified │                       │
        ▼                       ▼                       ▼
```

#### Supernode Requirements

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| **CPU** | 4 cores | 8+ cores |
| **RAM** | 4 GB | 8+ GB |
| **Bandwidth** | 10 Mbps up | 50+ Mbps up |
| **Uptime** | 4 hours/day | 12+ hours/day |
| **Storage** | 2 GB (model cache) | 10+ GB |

#### Delegation Protocol

1. **Capability Advertisement**: Supernodes broadcast signed `delegate_capability` announcement to DHT:
   ```json
   {
     "type": "delegate_capability",
     "peerID": "12D3KooW...",
     "services": ["embed", "ann_query", "sig_verify"],
     "rateLimit": {"requestsPerMinute": 10, "maxConcurrent": 5},
     "model": "Xenova/all-MiniLM-L6-v2 @sha256:abc123",
     "uptime": 0.95,
     "signature": "ed25519_signature_here"
   }
   ```

2. **Secure Request**: Low-tier peers encrypt requests with supernode's public key (from capability announcement) and sign with their own key:
   ```json
   {
     "type": "delegate_request",
     "requestID": "uuid_v4",
     "service": "embed",
     "payload": "encrypted_text_description",
     "requesterPubKey": "ed25519_public_key",
     "timestamp": 1741400000,
     "signature": "ed25519_signature_here"
   }
   ```

3. **Verifiable Response**: Supernodes return results with cryptographic proof:
   ```json
   {
     "type": "delegate_response",
     "requestID": "uuid_v4",
     "result": {
       "embedding": [0.12, -0.07, ...],
       "model": "Xenova/all-MiniLM-L6-v2 @sha256:abc123",
       "norm": 1.0
     },
     "supernodePubKey": "ed25519_public_key",
     "timestamp": 1741400005,
     "signature": "ed25519_signature_here"
   }
   ```

4. **Local Verification**: Requesting peer verifies:
   - Supernode signature matches advertised public key
   - Embedding norm ≈ 1.0 (±0.01 tolerance)
   - Model version matches expected canonical model
   - Request ID matches original request
   - Optional: Cross-check subset with local minimal model

#### Trust & Safety

- **No blind trust**: All delegated results are cryptographically signed and locally sanity-checked.
- **Reputation-weighted selection**: Peers prefer supernodes with high `uptime` and positive interaction history.
- **Sybil resistance**: Supernodes must maintain 7-day uptime history to be highly ranked; opt-in stake boosts visibility.
- **Privacy preserved**: Requests are E2E encrypted; supernodes see only channel descriptions—never raw chat content.
- **Graceful degradation**: If no supernodes available, Low/Minimal peers use smaller models or word-hash fallback.

#### Supernode Incentives

| Incentive Type | Description |
|----------------|-------------|
| **Reputation badges** | Visible "Trusted Supernode" badge in profile; boosts match priority |
| **Priority queuing** | Supernodes receive faster ANN results from other supernodes |
| **Optional tips** | Lightning Network micropayments for verified assistance (opt-in) |
| **Governance weight** | High-rep supernodes carry more weight in community moderation decisions |
| **Early feature access** | Beta features rolled out to active supernodes first |

#### Configuration

```javascript
// Peer config example
const peerConfig = {
  tier: 'low',
  delegation: {
    enabled: true,
    preferLocal: true,        // Try local first, delegate only if needed
    trustedSupernodes: [],    // Optional: manually pinned supernode peerIDs
    maxDelegationsPerMinute: 3,
    maxResponseLatencyMs: 5000
  },
  supernode: {
    enabled: false,           // Set true to serve others
    maxConcurrentRequests: 5,
    advertiseUptime: true
  }
};
```

#### Delegation Privacy Guarantees

- **Request encryption**: Delegation requests encrypted with supernode's public key; only intended supernode can decrypt.
- **No logging policy**: Supernodes expected to discard request contents after computing results. *Future work: ZK proofs of correct computation without revealing inputs.*
- **Minimal exposure**: Only channel descriptions (not chat messages) are delegated. Users can disable delegation for sensitive channels via Settings.
- **Auditability**: Supernodes publish signed uptime/throughput metrics; anomalous behavior can be flagged by community.
- **User control**: Delegation can be disabled per-channel or globally at any time.

#### Delegation Health Metric

Supernodes announce a `delegation_health` signal every 5 minutes:
```json
{
  "type": "delegation_health",
  "peerID": "12D3KooW...",
  "successRate": 0.98,
  "avgLatencyMs": 250,
  "requestsServed24h": 142,
  "timestamp": 1741400000,
  "signature": "ed25519_signature_here"
}
```

Peers use `successRate` and `avgLatencyMs` to select reliable supernodes. Metrics below 0.85 success rate trigger automatic deprioritization.

---

### 8. Threat Model

ISC operates under the following security assumptions:

| Threat | Assumption | Mitigation (Phase 1: Trusted) | Mitigation (Phase 2+: Public) |
|--------|------------|-------------------------------|-------------------------------|
| **Malicious supernodes** | Honest-but-curious; may log requests or return incorrect embeddings | Local sanity checks + trusted operator selection | + Reputation weighting + optional SNARK proofs (future) |
| **DHT bootstrap peers** | Not actively adversarial; may go offline | Multiple bootstrap peers; graceful reconnect; fallback to centralized rendezvous (opt-in) | Same |
| **Sybil attackers** | Can create many identities | Social trust barrier (invite-only) | Reputation decay + uptime history + opt-in stake |
| **Network eavesdroppers** | Can observe traffic patterns but not decrypt content | WebRTC DTLS + Noise protocol + E2E encryption for delegation requests | Same |
| **Browser compromise** | Out of scope (assumes honest client) | N/A — user responsible for browser security | Same |
| **Model poisoning** | Attacker distributes malicious embedding model | Canonical model registry (DHT-hosted, signed); clients reject unknown model hashes | Same |
| **Reputation gaming** | Attacker creates fake positive interactions | N/A (reputation not yet enabled) | Mutual signing + time-weighted decay |

**Explicitly Out of Scope**:
- Browser zero-day exploits
- User key compromise (private key stored in IndexedDB; user responsible for passphrase)
- Physical device theft
- Government-level traffic correlation attacks (Tor/I2P integration mitigates but doesn't eliminate)

---

## Device Tiers

ISC adapts to device RAM, CPU, and network at startup. A capability probe runs before any model loads — it measures `navigator.hardwareConcurrency`, a quick WASM microbenchmark, and network type — then selects a tier:

| Tier | Target | Embedding Model | Relations | ANN | Hashes | Refresh | **Delegation** | **Delegate Mode** |
|------|--------|-----------------|-----------|-----|--------|---------|----------------|-------------------|
| **High** | Desktop / modern laptop | `all-MiniLM-L6-v2` (384-dim, ~22 MB) | All (max 5) | HNSW via usearch | 20 | 5 min | ✅ Can request | ✅ Can serve (opt-in) |
| **Mid** | Mid-range phone / older laptop | `paraphrase-MiniLM-L3-v3` (384-dim, ~8 MB) | Root + 2 | HNSW lite | 12 | 8 min | ✅ Can request | ⚠️ Limited serve |
| **Low** | Budget phone / slow connection | `gte-tiny` (384-dim, ~4 MB) | Root only | Linear scan | 8 | 15 min | ✅ Can request | ❌ Cannot serve |
| **Minimal** | Very constrained / 2G | Word-hash fallback (no model) | Root only | Hamming | 6 | 20 min | ✅ Can request | ❌ Cannot serve |

> **Delegate Mode**: High-tier peers can opt into `supernode` mode via Settings. When enabled, they advertise delegation capabilities to the DHT and assist Low/Minimal peers with embedding, ANN, or verification tasks. All delegated work is cryptographically signed and locally verifiable—no blind trust required. See [Supernode Delegation Architecture](#7-supernode-delegation-architecture) for requirements.

**Other tier-aware behaviours:**

- **Workers**: High/Mid use a dedicated Web Worker pool (embed + ANN in background). Low runs embeddings inline. Minimal skips WASM entirely.
- **Candidate cap**: High fetches up to 100 DHT candidates per key; Low fetches 20 to conserve battery.
- **Monte Carlo samples**: Distribution matching uses 100 samples on High, 20 on Low.
- **Model caching**: Models are cached in IndexedDB after first download; subsequent loads skip the download entirely.
- **Tier override**: Users can manually up/downgrade via Settings.

```javascript
async function detectTier() {
  const cores = navigator.hardwareConcurrency ?? 2;
  const mem   = navigator.deviceMemory ?? 1;       // GB, where available
  const conn  = navigator.connection?.effectiveType ?? '4g';

  if (cores >= 4 && mem >= 4 && conn !== '2g') return 'high';
  if (cores >= 2 && mem >= 2)                   return 'mid';
  if (conn === '2g' || mem < 1)                 return 'minimal';
  return 'low';
}
```

---

## Tech Stack

| Layer | Library | Notes |
|---|---|---|
| **Embedding** | [`@xenova/transformers.js`](https://github.com/xenova/transformers.js) | Multiple models selectable by tier; ONNX/WASM, no server |
| **P2P network** | [`js-libp2p`](https://github.com/libp2p/js-libp2p) | Transports: WebSockets + WebRTC; muxer: yamux; encryption: noise |
| **DHT** | `@libp2p/kad-dht` | Kademlia for peer discovery and vector-hash storage |
| **ANN index** | [`usearch`](https://github.com/unum-cloud/usearch) (WASM) | HNSW for on-device approximate nearest-neighbor queries; linear fallback on Low tier |
| **Hashing** | Custom seeded LSH | Random-projection LSH with deterministic seed per channel for consistent bucket placement |
| **PubSub** | `@libp2p/pubsub` / floodsub | Group room announcements and social-layer events |
| **Crypto / Identity** | Web Crypto API | ed25519 keypair generation; sign all DHT payloads and posts |
| **State / channels** | `localStorage` + IndexedDB | Channel metadata in `localStorage`; embeddings, history, keypair in IndexedDB |
| **Distribution math** | Vanilla JS | Gaussian sampling, Monte Carlo distance, bipartite relational alignment |
| **UI** | Vanilla HTML/JS | (or React/Svelte — your choice) |

---

## Safety, Privacy & Authenticity

ISC achieves strong authenticity, safety, and privacy properties by building on its fully browser-based, P2P architecture. It starts with structural advantages over centralized platforms — no central server collecting data, no persistent profiles beyond user-controlled local storage, and ephemeral announcements (TTLs prevent long-term tracking).

The design is inspired by **Nostr** (cryptographic authenticity, censorship resistance, relay resilience), adopting its best elements while extending them with ISC's vector-space primitives.

### Authenticity

**Built-in (libp2p baseline):**
- libp2p peer IDs provide cryptographic identity (Noise protocol + ed25519 keys).
- All DHT announcements and WebRTC streams are authenticated via libp2p's built-in crypto.

**Nostr-inspired enhancements (implemented):**
- **User keypairs**: On first launch, an ed25519 keypair is generated via Web Crypto API. The public key is the user's persistent "ISC identity" (npub-like). The private key is stored in IndexedDB, optionally encrypted with a user passphrase.
- **Signed announcements**: Every `contentRouting.put` payload is signed with the private key. Peers verify signatures during ANN refinement and discard unsigned or tamper-evident candidates.
- **Signed posts**: All social-layer content (posts, reposts, reactions) carry a signature — tamperproof by design, no central authority required.

### Safety

**Structural baseline:**
- Ephemeral TTLs + no persistent profiles reduce long-term targeting.
- WebRTC DTLS E2E encryption for all chat streams.

**Deployment modes:**

ISC supports two deployment modes with different safety assumptions:

| Mode | Trust Model | Safety Mechanisms |
|------|-------------|-------------------|
| **Trusted Network** (Phase 1) | Pre-existing social trust (invite-only, communities, organizations) | Rate limiting + mute/block + semantic filters |
| **Public Network** (Phase 2+) | Open participation, potential adversaries | Reputation weighting + stake signaling + coherence checks + decentralised moderation |

**Implemented mechanisms (Trusted Network mode):**
- **Rate limiting**: Per-peer announcement caps (max 5 DHT puts/minute, 50/hour). Enforced client-side; supernodes verify and reject excess requests.
- **Mute / block lists**: Signed mute events stored in DHT (per user). Clients auto-filter flagged vectors from match results.
- **Semantic filters**: Sim-threshold controls (per channel) let users define their own "safe zone" — content far from their channel's vector space is naturally deprioritised.
- **Harassment exit**: Auto-decay chats when similarity drops below threshold (thought drift = natural exit). One-click mute with propagation.

**Planned mechanisms (Public Network mode — Phase 2):**
- **Reputation weighting**: Peers accumulate reputation via successful interactions. Low-rep announcements are deprioritized in ANN results. Reputation decays with 30-day half-life.
- **Stake-based signaling** (opt-in): Users may lock Lightning satoshis as sybil-resistance signal; slashed on verified abuse. *Never required for basic use.*
- **Semantic coherence checks**: Announcements with embeddings far from stated description (>0.6 cosine distance) are flagged for supernode review.
- **Mute/block propagation**: Signed mute events stored in DHT; high-rep peers carry more weight in filtering decisions.
- **Decentralised moderation**: Signed reports stored in DHT; clients weight them by reporter reputation. No central moderation team; safety emerges from network geometry.

### Privacy

**Built-in baseline:**
- No central servers; all data lives locally or traverses P2P.
- Only vectors + peerID announced publicly; raw text is never broadcast unless the user explicitly posts.
- Chats E2E encrypted.

**Enhanced capabilities:**
- **Ephemeral keys**: Optional throwaway keypairs per session or channel — no persistent npub linkage.
- **IP protection**: libp2p circuit relays + public STUN/TURN fallbacks obfuscate direct IPs. Optional Tor / I2P routing via community libp2p transport plugins for high-privacy users.
- **Metadata minimisation**: Vector-only announcements for discovery; raw text revealed only in direct WebRTC chats.
- **Plausible deniability**: Channel spread (σ) adds deliberate fuzz to announced position — exact thoughts are not recoverable from the vector alone.
- **Data sovereignty**: Users control full export/delete of all local data. No cross-session tracking without explicit follows.
- **Delegation privacy**: When using supernode delegation:
  - Requests are E2E encrypted with supernode's public key
  - Only channel descriptions are delegated (never chat messages)
  - Users can disable delegation per-channel via Settings
  - Supernodes expected to discard request contents after computation
  - Future: ZK proofs enable verification without revealing inputs

**Comparison:**

| Aspect | ISC | Nostr | X (centralized) |
|---|---|---|---|
| **Authenticity** | Keypair signing + libp2p | Keypair signing | Platform verification |
| **Safety** | Layered anti-spam + semantic filters + mutes | Client-side + propagation | Central bans + biases |
| **Privacy** | No servers; optional Tor; ephemeral keys | Public-by-default; Tor mitigable | Full surveillance |
| **Censorship resistance** | DHT + WebRTC; no deplatforming | Relay-based; resilient | Platform-controlled |

---

## Social Network Layer

ISC's P2P and semantic foundations support a full decentralised social network — achieving parity with X (Twitter) while exceeding it through geometry-native interactions. All data (posts, profiles, reactions) is stored and queried via DHT with TTLs for ephemerality; embeddings make feeds explainable and serendipitous by design.

**Indicative timeline**: Q2 2026 — Posts & feeds · Q3 2026 — Interactions & DMs · Q4 2026 — Communities & monetisation · 2027 — Semantic innovations.

### Posts & Feeds

- **Posts**: Users embed text (and media URLs via IPFS) into the space, announce via LSH-hashed keys in DHT with a timestamp and channel ID. 280-char short posts and threaded long-form articles are both supported.
- **For You feed**: Semantic proximity feed — ranked ANN queries on active channels surface nearby posts (cosine sim > 0.6).
- **Following feed**: Posts from explicitly followed peers, tracked in local IndexedDB.
- **Explainability**: Every ranked post shows its similarity score and which channel it matched ("This post is 0.82 similar to your 'AI ethics' channel").
- **Global Explore**: Aggregate high-engagement clusters surface trending vector clouds without central curation.

### Interactions

- **Likes**: Lightweight DHT announcements `{peerID, postID, "like"}`.
- **Reposts**: Re-announce with your own vector — shifts propagation toward your distribution for hybrid reach.
- **Replies**: Threaded WebRTC streams or DHT-linked posts.
- **Quotes**: Embed original + your commentary as a fused vector.
- **Trending**: Aggregate engagement counts via PubSub; dense clusters signal viral topics.

### Profiles & Follows

- **Profile**: Aggregated message of a peer's channel distributions (bio as mean vector). No central verification needed; reputation emerges from mutual proximity and interaction history.
- **Follow**: Subscribe to a peer's channel distributions via libp2p pubsub.
- **Suggested follows**: Ranked by ANN queries on your active channels.
- **Web of Trust**: Mutual follows accumulate reputation scores; high-rep peers carry more weight in mute/flag propagation.

### Communities

- **Shared channels**: Groups of peers co-edit a channel's mean/spread, creating semantic "neighbourhoods" announced via DHT.
- **Audio Spaces**: Mesh broadcast within a dense channel cluster (WebRTC audio).
- **DMs**: 1:1 or group WebRTC streams, E2E encrypted via libsodium. Extended to video calls for full X parity.
- **Semantic moderation**: Off-vector posts are naturally deprioritised; signed reports route to community moderators.

### Exceeding Centralized Platforms

| Feature | ISC advantage |
|---|---|
| **Explainable feeds** | Similarity scores visible; no opaque algorithm |
| **Echo chamber mitigation** | Chaos mode perturbs distribution for cross-topic serendipity |
| **Thought bridging** | Local AI suggests replies that geometrically bridge two vectors |
| **Vibe rooms** | Proximity chats auto-form without invites; exit naturally as you drift |
| **Places** | Idea boards where posts evolve into projects — vectors as editable graph nodes |
| **Fuzzy anonymity** | Match on vectors without revealing peerID |
| **No ads** | Monetisation via crypto micropayments / Lightning Network tips (opt-in) |
| **Portability** | Export distributions; interop with Bluesky / AT Protocol |
| **Resilience** | No deplatforming; self-healing peer-relay for temporarily offline users |

---

## Getting Started

> **No build step required for the MVP** — the app is a single HTML file importable via `<script type="module">`.

1. **Clone the repo**
   ```bash
   git clone https://github.com/yourname/isc.git
   cd isc
   ```

2. **Serve locally** (any static server works)
   ```bash
   npx serve .
   # or
   python3 -m http.server 8080
   ```

3. **Open** `http://localhost:8080` in two browser tabs (or two separate browsers).

4. In each tab:
   - Create or pick a **channel** (a default channel is created on first launch)
   - Type a description of your current thoughts — optionally add **relation tags** (location, time, mood…)
   - Click **Embed** — the model runs locally, producing your distribution
   - Click **Find Matches** — peers at nearby positions appear, ranked by similarity
   - Click a match to open a 1:1 chat, or let dense clusters auto-mesh into a group

> **First run**: the embedding model (~4–22 MB depending on tier) is downloaded and cached in IndexedDB. Subsequent loads skip the download entirely.

> **Testing locally**: open two tabs, wait ~10 s for DHT bootstrap, then embed similar text in both. They should surface each other as a match.

### Testing Supernode Delegation Locally

1. **Start a supernode**:
   ```bash
   # In one terminal
   npx serve . --port 8080
   # Open http://localhost:8080?tier=high&supernode=true
   # Enable "Supernode Mode" in Settings → Delegation
   ```

2. **Start a low-tier client**:
   ```bash
   # In another terminal
   npx serve . --port 8081
   # Open http://localhost:8081?tier=low&delegate=true
   # Enable "Use Delegation" in Settings → Delegation
   ```

3. **Observe delegation**:
   - The low-tier client requests embedding assistance from the supernode
   - Browser DevTools → Network tab shows encrypted delegation messages (protocol: `/isc/delegate/1.0`)
   - Both peers should match semantically and form a chat within 30 seconds
   - Verify in DevTools Console: `peer.delegation.stats` shows request/response counts

4. **Test fallback**:
   - Close the supernode tab
   - Low-tier client should gracefully degrade to local minimal model
   - Match quality decreases but functionality remains

> **Note**: Local testing uses self-signed keys. Production delegation requires proper key management, reputation bootstrapping, and NAT traversal configuration.

---

## Roadmap

### Phase 1: Core Reliability (Q1–Q2 2026)

**Target deployment**: Trusted Networks (invite-only, private communities)

Foundation: semantic matching + peer discovery + delegation infrastructure.

- [ ] MVP — single-channel semantic matching + 1:1 WebRTC chat
- [ ] Basic multi-channel UI — create, switch, manage multiple channels
- [ ] Supernode delegation protocol + capability advertisement
- [ ] Layered anti-spam (rate limits only; reputation/stake deferred to Phase 2)
- [ ] Model version negotiation + compatibility shards
- [ ] Device tier auto-detection + delegate mode UI
- [ ] Threat model validation + security audit (community)
- [ ] NAT traversal improvements (circuit relay pool)

**Success criteria**: 50+ concurrent users; <5% connection failure rate; <10s median time-to-first-match.

### Phase 2: Scale & Safety (Q3–Q4 2026)

**Target deployment**: Federated Networks → Public Networks

Hardening: relational embeddings + reputation system + moderation primitives.

- [ ] Relational embeddings — cross-channel semantic composition
- [ ] Reputation system + signed moderation events (for public network mode)
- [ ] Offline queue + reconnect logic
- [ ] Delegation health metrics + supernode ranking
- [ ] PWA — installable on mobile, offline-capable shell
- [ ] IPFS deployment — zero-infra hosting for static bundle
- [ ] Community model registry + migration tooling

**Success criteria**: 1,000+ DAU; <10% delegation failure rate; sustainable supernode participation (10% of High-tier peers).

### Phase 3: Social Layer (2027)

**Target deployment**: Public Networks (open participation)

Features: posts, feeds, communities — built on reliable foundation.

- [ ] Posts & semantic feeds ("For You" + "Following")
- [ ] Reactions (likes, reposts, replies, quotes)
- [ ] Profiles & follow / Web of Trust
- [ ] Communities — shared channel distributions
- [ ] Audio Spaces (WebRTC mesh audio)
- [ ] Video calls (WebRTC, parity with X)
- [ ] Chaos mode — random perturbation for serendipitous cross-topic matches
- [ ] Crypto tipping / Lightning Network (opt-in monetisation) — *enables stake-based signaling*
- [ ] ZK proximity proofs — prove sim > threshold without revealing exact vector

**Success criteria**: 10,000+ DAU; <1% critical error rate; positive supernode economics (tips cover infrastructure costs for 20% of supernodes).

### Phase 4: Ecosystem (2028+)

Expansion: interop, innovation, sustainability.

- [ ] AT Protocol / Bluesky interop
- [ ] Mobile native apps (React Native / Flutter)
- [ ] Advanced moderation tools (community courts)
- [ ] DAO governance for protocol upgrades
- [ ] Enterprise deployment options (private ISC instances)

---

## Contributing

Pull requests, ideas, and experiments welcome. This is an early-stage prototype — the goal is to learn what emerges when people navigate by thought rather than by URL.

### Security Review Checklist

Before merging delegation-related or cryptographic PRs, ensure:

- [ ] All delegated responses are cryptographically signed (ed25519)
- [ ] Local verification logic has unit tests for edge cases (invalid sigs, malformed embeddings, timeout)
- [ ] Rate limiting is enforced on both request and response paths
- [ ] Fallback behavior is tested (no supernodes available, network partition)
- [ ] Memory usage is bounded (no unbounded request queues; max 100 pending delegations)
- [ ] Encryption keys are never logged or exposed in DevTools
- [ ] Model version checks prevent cross-model similarity computation
- [ ] Reputation events require mutual signing (both parties confirm interaction)
- [ ] DHT announcements include TTL and are signed
- [ ] WebRTC streams use DTLS encryption (verify via browser DevTools)

PRs failing any checklist item will be rejected without review.

1. Fork the repo
2. Create a feature branch: `git checkout -b feature/my-idea`
3. Commit your changes: `git commit -m 'Add my idea'`
4. Push and open a PR

Please keep PRs focused; large changes should be discussed in an issue first.

---

## License

MIT © 2025 ISC Contributors
