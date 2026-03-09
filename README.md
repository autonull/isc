# ISC — Idea Space Chat

> *Find your people by the shape of your thoughts.*

A fully **decentralized, browser-only** chat platform that uses lightweight in-browser LLM embeddings to place your ideas in a shared vector space — then serendipitously connects you with peers whose mental distributions are closest to yours. No servers. No accounts. No surveillance. Just thought and conversation.

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](#license)
[![Browser-only](https://img.shields.io/badge/runs%20in-browser-brightgreen)](#tech-stack)
[![P2P](https://img.shields.io/badge/network-libp2p%20%2B%20WebRTC-orange)](#tech-stack)

---

## Table of Contents

- [Overview](#overview)
- [How It Works](#how-it-works)
- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
  - [Peer Initialization](#1-peer-initialization)
  - [Generating Your Distribution](#2-generating-your-distribution)
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
| **Fuzzy identity** | You're represented as a *distribution* in embedding space, not a fixed point |
| **Ephemeral by default** | Announcements expire (TTL-based); no persistent profile |
| **E2E encrypted** | WebRTC's built-in DTLS secures all chat streams |

---

## How It Works

```
You type your thoughts
        │
        ▼
  [transformers.js]          ← runs entirely in your browser
  384-dim embedding vector
        │
        ▼
  LSH → DHT keys             ← vector hashed into ~20 Kademlia keys
        │
        ▼
  Announce to DHT            ← {peerID, vector, TTL} stored at each key
        │
        ▼
  Query DHT for matches      ← pull candidate peers from overlapping buckets
        │
        ▼
  Local ANN re-rank          ← HNSW (usearch) refines by true cosine sim
        │
        ▼
  Dial top-k peers           ← WebRTC DataChannel opened via libp2p
        │
        ▼
  Chat in a mesh group       ← floodsub / direct DataChannels
```

The key insight: rather than routing by topic *labels*, ISC routes by *semantic geometry*. Two people thinking about "the ethics of AI art" and "copyright in machine creativity" will find each other even if they used completely different words.

---

## Tech Stack

| Layer | Library | Notes |
|---|---|---|
| **Embedding** | [`@xenova/transformers.js`](https://github.com/xenova/transformers.js) | `all-MiniLM-L6-v2` — 384-dim, ~20 MB, <100 ms/inference via ONNX/WASM |
| **P2P network** | [`js-libp2p`](https://github.com/libp2p/js-libp2p) | Transports: WebSockets + WebRTC; muxer: yamux; encryption: noise |
| **DHT** | `@libp2p/kad-dht` | Kademlia for peer discovery and vector-hash storage |
| **ANN index** | [`usearch`](https://github.com/unum-cloud/usearch) (WASM) | HNSW for on-device approximate nearest-neighbor queries |
| **Hashing** | Custom LSH | Simple random-projection LSH, ~50 lines of JS |
| **PubSub** | `@libp2p/pubsub` / floodsub | Optional group room announcements |
| **Distribution math** | Vanilla JS | Gaussian sampling (mean μ, covariance Σ), Monte Carlo distance |
| **UI** | Vanilla HTML/JS | (or React/Svelte — your choice) |

> **Browser constraints**: All heavy compute (embeddings, ANN) runs in Web Workers to keep the UI thread responsive.

---

## Architecture

### 1. Peer Initialization

When a user loads the static app (hostable on IPFS or GitHub Pages), a libp2p node is spun up in-browser:

```js
import { createLibp2p } from 'libp2p';
import { webSockets }   from '@libp2p/websockets';
import { webRTC }       from '@libp2p/webrtc';
import { noise }        from '@chainsafe/libp2p-noise';
import { yamux }        from '@chainsafe/libp2p-yamux';
import { kadDHT }       from '@libp2p/kad-dht';
import { bootstrap }    from '@libp2p/bootstrap';

const node = await createLibp2p({
  transports:          [webSockets(), webRTC()],
  connectionEncryption:[noise()],
  streamMuxers:        [yamux()],
  peerDiscovery:       [bootstrap({ list: ['/ip4/.../tcp/.../p2p/Qm...'] })],
  services: { dht: kadDHT({ kBucketSize: 20 }) },
});
await node.start();
```

Bootstrap peers can be any well-known libp2p relay (public ones exist in the libp2p docs, or run your own seed tab).

---

### 2. Generating Your Distribution

Users enter free-form text describing their current thoughts. Each entry is embedded locally:

```js
import { pipeline } from '@xenova/transformers';

const extractor = await pipeline('feature-extraction', 'Xenova/all-MiniLM-L6-v2');
const output = await extractor(text, { pooling: 'mean', normalize: true }); // Float32Array, 384D
```

For multiple inputs (e.g., "AI ethics, cyberpunk art, quantum computing"), ISC computes a **multivariate Gaussian** (mean μ, sampled variance Σ), representing a *fuzzy* presence in the space rather than a single fixed point. Users can tune a "spread" slider to broaden or tighten their distribution.

Matching strategies (selectable):
- **Point match** — sample a single point from distribution (fast, good for MVP)
- **Distribution match** — approximate Earth Mover's Distance / KL divergence via Monte Carlo (richer, slower)

---

### 3. Announcing to the DHT

Vectors are mapped to DHT keys via **Locality-Sensitive Hashing (LSH)**:

```js
function lshHash(vec, numHashes = 20, hashLen = 32) {
  return Array.from({ length: numHashes }, () => {
    const proj = vec.map(() => Math.random() * 2 - 1); // random hyperplane
    return vec
      .map((v, j) => (v * proj[j] > 0 ? '1' : '0'))
      .join('')
      .slice(0, hashLen);
  });
}

const hashes = lshHash(userEmbedding);
for (const key of hashes) {
  await node.contentRouting.put(
    key,
    JSON.stringify({ peerID: node.peerId.toString(), vec: Array.from(userEmbedding), ttl: 3600 })
  );
}
```

Announcements carry a 1-hour TTL and are refreshed every 5–10 minutes as thoughts evolve.

---

### 4. Querying for Proximals

On "Find my people", the current distribution is hashed and each key is queried:

```js
const candidates = [];
for (const key of lshHash(userEmbedding)) {
  const values = await node.contentRouting.getMany(key, { count: 50 });
  candidates.push(...values.map(v => JSON.parse(v)));
}
```

Candidates are then **refined locally** using usearch (HNSW) by true cosine similarity, filtering to top-k (default 10–20) above a threshold (default 0.7). For distribution-to-distribution matching, distances are averaged over 100 Monte Carlo samples.

---

### 5. Forming Chats

Top matches are dialed directly over WebRTC via a custom libp2p protocol:

```js
const stream = await node.dialProtocol(peerID, '/isc/chat/1.0');
stream.write(JSON.stringify({ msg: 'Hey, our thoughts are proximal!' }));
```

**Group chats**: when 3+ peers match, each dials the others into a mesh. Libp2p's floodsub (or DataChannels with broadcast) handles message delivery. Rooms are identified by hashing the group's shared vector centroid — announced via DHT so latecomers can join.

The UI presents chat sessions as floating bubbles tagged with the semantic topic that drew peers together.

---

### 6. Handling Dynamics

| Challenge | Approach |
|---|---|
| **Peer churn** | DHT naturally handles leaves; heartbeat pings drop dead chat partners |
| **Cold start** | Bootstrap via public libp2p relays; "seed tab" pattern for early adopters |
| **Scale (1k+ peers)** | LSH + DHT approximates well; add gossip (peers share neighbor lists) for > 10k |
| **Thought drift** | Users can "wander" by updating their distribution; stale TTLs auto-expire |
| **NAT traversal** | Public STUN/TURN servers as fallback; libp2p circuit relays for restricted NATs |

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
   - Enter your current thoughts in the text box
   - Click **Embed** — your ideas are vectorized locally
   - Click **Find Proximals** — peers at nearby positions in idea space appear
   - Click a match to open a chat

> The first run downloads the `all-MiniLM-L6-v2` ONNX model (~20 MB) and caches it in the browser's IndexedDB. Subsequent loads are instant.

---

## Roadmap

- [ ] **MVP** — single-page app: embed → announce → query → chat
- [ ] **Distribution UI** — sliders for spreading your presence across multiple interest axes
- [ ] **2D visualization** — PCA/UMAP projection of live peer cloud
- [ ] **Chaos mode** — random distribution perturbation for serendipitous cross-topic matches
- [ ] **Audio** — WebRTC voice in group rooms
- [ ] **Persistence** — optional local history via IndexedDB
- [ ] **Spam resistance** — lightweight Proof-of-Work on DHT announcements
- [ ] **Mobile** — Progressive Web App wrapper
- [ ] **IPFS hosting** — deploy the static bundle to IPFS for zero-infra distribution

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
