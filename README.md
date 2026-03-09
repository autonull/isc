# ISC — Internet Semantic Chat

> *Meet your thought neighbors.*

A fully **decentralized, browser-only** chat platform that uses lightweight in-browser LLM embeddings to place your ideas in a shared vector space — then serendipitously connects you with peers whose mental distributions are closest to yours. No servers. No accounts. No surveillance. Just thought and conversation.

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](#license)
[![Browser-only](https://img.shields.io/badge/runs%20in-browser-brightgreen)](#tech-stack)
[![P2P](https://img.shields.io/badge/network-libp2p%20%2B%20WebRTC-orange)](#tech-stack)

---

## Table of Contents

- [Overview](#overview)
- [Channels](#channels)
- [How It Works](#how-it-works)
- [Device Tiers](#device-tiers)
- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
  - [Peer Initialization](#1-peer-initialization)
  - [Channels & Distributions](#2-channels--distributions)
  - [Announcing to the DHT](#3-announcing-to-the-dht)
  - [Querying for Proximals](#4-querying-for-proximals)
  - [Forming Chats](#5-forming-chats)
  - [Handling Dynamics](#6-handling-dynamics)
- [Getting Started](#getting-started)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

ISC is an experiment in **semantic proximity networking**. Instead of joining a chat room by topic tag or URL, you describe your current thoughts — and the system finds other people thinking similar things right now.

Key properties:

| Property | Detail |
|---|---|
| **Fully browser-native** | Zero server-side compute; all embedding and routing runs in-browser |
| **Serverless P2P** | Kademlia DHT for discovery, WebRTC DataChannels for chat |
| **Channels** | Maintain multiple named presence contexts and switch between them instantly |
| **Adaptive by device** | Model size, Worker concurrency, and matching depth auto-scale to device capability |
| **Fuzzy identity** | You're represented as a *distribution* in embedding space, not a fixed point |
| **Ephemeral by default** | Announcements expire (TTL-based); no persistent profile |
| **E2E encrypted** | WebRTC's built-in DTLS secures all chat streams |

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

- Each channel has a **name**, a **description** (free-form text, the seed of your embedding), and an optional **spread** value (how wide your distribution is in the space).
- Channels are **stored locally** in `localStorage` / IndexedDB — they persist across sessions and survive browser refresh.
- When a channel is **active**, its distribution is announced to the DHT and refreshed on a timer. Inactive channels are silently withdrawn (TTL expiry) so you don't accumulate ghost presences.
- You can **edit** a channel's description at any time; the embedding is recomputed and the DHT announcement is updated within seconds.
- Channels are **independent**: switching to "Evening" does not collapse your "Work" chats — in-progress conversations stay live until you explicitly close them or they time out.
- Each channel maintains its own **match list** and **chat history** (session-local; optionally persisted).

---

## How It Works

```
You pick a channel (or create one)
        │
        ▼
  Channel description is embedded      ← transformers.js, runs in your browser
  → 384-dim vector (or lower, on weak devices)
        │
        ▼
  Distribution built around vector     ← mean μ, spread Σ (tunable per channel)
        │
        ▼
  LSH → Kademlia keys                  ← seeded projections for reproducibility
        │
        ▼
  Announce to DHT                      ← {peerID, channelID, vec, TTL} per key
        │
        ▼
  Query DHT for matches                ← overlapping buckets → candidate list
        │
        ▼
  Dedupe + local ANN re-rank           ← HNSW (usearch/WASM) by cosine sim
        │
        ▼
  Dial top-k peers via WebRTC          ← /isc/chat/1.0 over libp2p stream
        │
        ▼
  Chat in a mesh group                 ← floodsub or direct DataChannels
```

The key insight: rather than routing by topic *labels*, ISC routes by *semantic geometry*. Two people thinking about "the ethics of AI art" and "copyright in machine creativity" will find each other even if they used completely different words.

---

## Device Tiers

ISC adapts to the device's RAM, CPU, and network at startup. A capability probe runs before any model loads — it measures `navigator.hardwareConcurrency`, a quick WASM microbenchmark, and network type — then selects a tier:

| Tier | Target | Embedding model | ANN | Hashes | Refresh |
|---|---|---|---|---|---|
| **High** | Desktop / modern laptop | `all-MiniLM-L6-v2` (384-dim, ~20 MB) | HNSW via usearch | 20 | 5 min |
| **Mid** | Mid-range phone / older laptop | `paraphrase-MiniLM-L3-v2` (384-dim, ~7 MB) | HNSW lite | 12 | 8 min |
| **Low** | Budget phone / slow connection | `gte-tiny` (384-dim, ~4 MB) | Linear scan | 8 | 15 min |
| **Minimal** | Very constrained / 2G | Predefined word-hash fallback (no model) | Hamming | 6 | 20 min |

**Other tier-aware behaviours:**

- **Workers**: High/Mid use a dedicated Web Worker pool (embed + ANN in background). Low runs embeddings inline (small model, acceptable). Minimal skips WASM entirely.
- **Candidate cap**: High fetches up to 100 DHT candidates per key; Low fetches 20 to conserve battery and bandwidth.
- **Monte Carlo samples**: Distribution-to-distribution matching uses 100 samples on High, 20 on Low.
- **Model caching**: Models are cached in IndexedDB after first download; on subsequent loads, the model is read from cache with zero network cost.
- **Tier override**: Users can manually up/downgrade via Settings if the auto-detected tier is wrong.

```js
// Simplified capability detection
async function detectTier() {
  const cores = navigator.hardwareConcurrency ?? 2;
  const mem   = navigator.deviceMemory ?? 1; // GB, where available
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
| **PubSub** | `@libp2p/pubsub` / floodsub | Group room announcements |
| **State / channels** | `localStorage` + IndexedDB | Channel metadata in `localStorage`; embeddings + history in IndexedDB |
| **Distribution math** | Vanilla JS | Gaussian sampling, Monte Carlo distance; tier-scaled sample count |
| **UI** | Vanilla HTML/JS | (or React/Svelte — your choice) |

---

## Architecture

### 1. Peer Initialization

When a user loads the static app (hostable on IPFS or GitHub Pages), the client:

1. Runs the capability probe → selects a device tier
2. Loads and caches the appropriate embedding model
3. Restores saved channels from `localStorage`
4. Spins up the libp2p node

```js
import { createLibp2p } from 'libp2p';
import { webSockets }   from '@libp2p/websockets';
import { webRTC }       from '@libp2p/webrtc';
import { noise }        from '@chainsafe/libp2p-noise';
import { yamux }        from '@chainsafe/libp2p-yamux';
import { kadDHT }       from '@libp2p/kad-dht';
import { bootstrap }    from '@libp2p/bootstrap';

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

Channels are plain objects persisted to `localStorage`:

```js
// Channel schema
{
  id:          'ch_work_7f3a',       // stable random ID
  name:        'Work',
  description: 'distributed systems, consensus algorithms',
  spread:      0.15,                 // Gaussian σ; 0 = point, 1 = very broad
  createdAt:   1741400000,
  updatedAt:   1741450000,
}
```

When activated, the description is embedded and a distribution constructed:

```js
import { pipeline } from '@xenova/transformers';

const extractor = await pipeline('feature-extraction', MODEL_FOR_TIER);
const raw = await extractor(channel.description, { pooling: 'mean', normalize: true });
const mu  = Array.from(raw.data);                        // 384-dim mean
const sig = channel.spread;                              // scalar spread

// Sample a point from the distribution for this announcement cycle
const sample = mu.map(v => v + sig * gaussianNoise());
```

Users can edit a channel description inline; the embedding recomputes and the DHT entry updates automatically. Channels can be **reordered**, **duplicated** (to fork an idea), or **archived** (withdrawn from DHT but kept locally).

Matching strategies (selectable per channel):
- **Point match** — single sampled point (fast; good for Low/Min tier)
- **Distribution match** — approximate EMD / KL via Monte Carlo (richer; High/Mid tier)

---

### 3. Announcing to the DHT

Vectors are mapped to DHT keys via **seeded Locality-Sensitive Hashing (LSH)**. The seed is derived from the channel ID so that the same channel always maps to the same bucket family — improving stability across reconnects:

```js
function lshHash(vec, channelId, numHashes, hashLen = 32) {
  const rng = seededRng(channelId); // deterministic from channel ID
  return Array.from({ length: numHashes }, () => {
    const proj = vec.map(() => rng() * 2 - 1);
    return vec.map((v, j) => (v * proj[j] > 0 ? '1' : '0')).join('').slice(0, hashLen);
  });
}

for (const key of lshHash(sample, channel.id, TIER.numHashes)) {
  await node.contentRouting.put(key, JSON.stringify({
    peerID:    node.peerId.toString(),
    channelID: channel.id,
    vec:       sample,               // the sampled point announced this cycle
    ttl:       TIER.ttl,
  }));
}
```

- The active channel's announce loop runs on `TIER.refreshInterval`.
- Inactive channels are not announced; their keys expire naturally via TTL.
- On connectivity loss (detected via `navigator.onLine` + libp2p events), the announce loop pauses and resumes automatically on reconnect.

---

### 4. Querying for Proximals

On "Find matches" for the active channel:

```js
const seen    = new Set();
const candidates = [];

for (const key of lshHash(currentSample, channel.id, TIER.numHashes)) {
  const values = await node.contentRouting.getMany(key, { count: TIER.candidateCap });
  for (const v of values) {
    const peer = JSON.parse(v);
    if (peer.peerID === node.peerId.toString()) continue; // skip self
    if (seen.has(peer.peerID)) continue;                  // deduplicate
    seen.add(peer.peerID);
    candidates.push(peer);
  }
}
```

Candidates are then **refined locally** using usearch (HNSW) by cosine similarity, filtering to top-k (default 10–20) above a configurable threshold (default 0.7). On Low/Minimal tier, a linear scan over the smaller candidate set replaces HNSW.

For distribution-to-distribution matching (High/Mid), distances are averaged over `TIER.mcSamples` Monte Carlo samples from both distributions.

---

### 5. Forming Chats

Top matches are dialed directly over WebRTC via a custom libp2p protocol:

```js
const stream = await node.dialProtocol(peerID, '/isc/chat/1.0');
stream.write(JSON.stringify({
  channelID: channel.id,
  msg:       'Hey, our thoughts are proximal!',
}));
```

**Group chats**: when 3+ peers match within the same channel, each dials the others into a mesh. Libp2p's floodsub handles message broadcast. Rooms are identified by hashing the group's shared vector centroid — announced via DHT so latecomers can join.

**Reconnect logic**: if a peer's stream drops, ISC waits 5 s then attempts a single reconnect dial. If that fails, the peer is marked as lost and the UI notifies the group. A full re-query finds fresh replacements.

The UI presents chat sessions grouped by channel, with floating chat bubbles tagged with the semantic topic that drew peers together.

---

### 6. Handling Dynamics

| Challenge | Approach |
|---|---|
| **Peer churn** | DHT naturally handles leaves; heartbeat pings every 30 s drop dead partners |
| **Cold start** | Bootstrap via public libp2p relays; "seed tab" pattern for early adopters |
| **Scale (1k+ peers)** | LSH + DHT approximates well; gossip (peers share neighbor lists) for > 10k |
| **Thought drift** | Edit a channel description → new embedding pushed to DHT within seconds |
| **NAT traversal** | Public STUN/TURN servers as fallback; libp2p circuit relays for hard NATs |
| **Offline / flaky net** | Announce loop pauses on `offline` event; resumes on `online`; in-flight chats queue messages |
| **Stale candidates** | TTL on all DHT entries; `updatedAt` timestamp in payload lets client discard old results |
| **Duplicate peers** | Client-side deduplication by `peerID` before ANN ranking |
| **Channel collision** | Channel IDs are random; per-channel LSH seeds prevent cross-channel bucket contamination |

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
   - Create or pick a **channel** (default channel created on first launch)
   - Type a description of your current thoughts — click **Embed**
   - Click **Find Matches** — peers at nearby positions appear
   - Click a match to open a chat

> **First run**: the embedding model (~4–20 MB depending on tier) is downloaded and cached in IndexedDB. Subsequent loads skip the download entirely.

> **Testing locally**: open two tabs, wait ~10 s for DHT bootstrap, then embed similar text in both. They should surface each other as a match.

---

## Roadmap

**Core**
- [ ] MVP — single-page app: channel → embed → announce → query → chat
- [ ] Channel UI — create, rename, edit, reorder, archive
- [ ] Device tier auto-detection + manual override in Settings

**Matching**
- [ ] Distribution UI — per-channel spread slider
- [ ] Distribution-to-distribution matching (EMD / Monte Carlo)
- [ ] 2D visualisation — PCA/UMAP projection of live peer cloud per channel

**Reliability**
- [ ] Reconnect / stream-recovery logic
- [ ] Offline queue — buffer messages sent while temporarily offline
- [ ] Seeded LSH — deterministic bucket placement per channel ID

**Features**
- [ ] Chaos mode — random perturbation for serendipitous cross-topic matches
- [ ] Audio — WebRTC voice in group rooms
- [ ] Local history — optional session persistence via IndexedDB

**Security & scale**
- [ ] Spam resistance — lightweight PoW on DHT announcements
- [ ] Reputation — flag/mute peers; propagated in-band
- [ ] Gossip KNN — peers share neighbour lists for > 10k scale

**Distribution**
- [ ] PWA — installable on mobile, offline-capable shell
- [ ] IPFS deployment — zero-infra hosting for the static bundle

---

## Contributing

Pull requests, ideas, and experiments welcome. This is an early-stage prototype — the goal is to learn what emerges when people navigate by thought rather than by URL.

1. Fork the repo
2. Create a feature branch: `git checkout -b feature/my-idea`
3. Commit your changes: `git commit -m 'Add my idea'`
4. Push and open a PR

Please keep PRs focused; large changes should be discussed in an issue first.

---

## License

MIT © 2025 ISC Contributors
