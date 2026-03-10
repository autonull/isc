# ISC System Feasibility Evaluation

## 1. Architectural Gaps & Network Realities

### Browser Backgrounding and Lifecycle

- **Issue**: Modern browsers aggressively throttle or suspend background tabs
  to save battery and memory. A "browser-only P2P" network relying on
  persistent DHT participation and WebRTC connections will face severe churn
  when users switch tabs or minimize the browser.
- **Impact**: DHT announcements will drop, and connections will be severed
  prematurely, breaking the continuous "P2P network" assumption. The "Seed Tab"
  pattern is a workaround, but requires active user management. Mobile browsers
  (iOS Safari, Android Chrome) kill background WebRTC streams and WebSocket
  connections even faster than desktop browsers, making a mobile-first P2P
  network in the browser practically infeasible without a native app component.

### WebRTC Signaling and NAT Traversal

- **Issue**: WebRTC requires an out-of-band signaling mechanism to exchange SDP
  offers/answers before a P2P connection can be established. The docs mention
  Libp2p circuit relays and public STUN/TURN, but don't clearly explain how the
  initial SDP exchange happens securely and reliably purely via the DHT without
  a centralized signaling server or persistent connection to a relay.
- **Impact**: If signaling relies on DHT puts/gets, the latency (often multiple
  seconds) will make chat initiation painfully slow. If it relies on circuit
  relays, the network centralization simply shifts to the relay operators,
  introducing potential bottlenecks and single points of failure.

### Supernode DHT Views & Data Consistency

- **Issue**: High-tier peers and Supernodes are expected to maintain a
  "persistent, continuously updated HNSW index of the global network state" by
  monitoring the DHT. However, a Kademlia DHT does not broadcast every
  key-value pair to every node. A single node only sees traffic destined for
  its own keyspace neighborhood.
- **Impact**: Supernodes cannot build a "global network state" passively. They
  would have to aggressively crawl the entire DHT (which is spammy and slow) or
  the protocol must be modified to use a PubSub architecture for global state
  (which doesn't scale well to high churn networks). The assumption that a
  supernode has a globally accurate HNSW index is mathematically flawed under
  standard Kademlia routing.

## 2. Mathematical & Algorithmic Concerns

### LSH (Locality-Sensitive Hashing) Collision Probabilities

- **Issue**: The system maps 384-dimensional dense vectors to DHT keys using
  random-projection LSH. Random projection preserves cosine similarity, but the
  "curse of dimensionality" means that preserving tight thresholds (e.g., >0.85
  similarity) in 384D space requires a massive number of hash functions or bits
  to avoid false positives (collisions of dissimilar vectors) and false
  negatives (missing similar vectors).
- **Impact**: If `numHashes` is small (e.g., 20 as specified for High tier),
  the recall for nearest neighbors in the DHT will be very low (high false
  negative rate). If it's large, the DHT put/get amplification factor becomes a
  denial-of-service vector. The tuning of these parameters to achieve
  acceptable recall without network flooding is highly non-trivial.

### Model Fragmentation

- **Issue**: Cosine similarity is only meaningful within the exact same
  embedding space. The protocol includes `modelHash` in the DHT key to isolate
  spaces.
- **Impact**: As models evolve or new ones are introduced, the network
  splinters into isolated shards. Users on `v1` cannot discover users on `v2`.
  The "90-day dual-announcement" mitigation requires clients to download and
  run multiple LLMs simultaneously, violating the browser resource constraints
  (especially for low/mid-tier devices).

### Relational Matching and Monte Carlo Sampling

- **Issue**: To handle "distribution fuzziness" (spread/sigma), the protocol
  uses Monte Carlo sampling (e.g., 100 samples per match) to calculate expected
  cosine similarity.
- **Impact**: This is computationally extremely expensive to do for every
  candidate peer discovered from the DHT. Doing `100 samples * 100 candidates =
  10,000` cosine similarity calculations in JavaScript on every UI refresh will
  block the main thread or heavily tax the Web Worker, draining battery. There
  are closed-form analytical approximations for the expected cosine similarity
  of two isotropic Gaussian distributions that would be orders of magnitude
  faster.

## 3. Security, Privacy & Trust Concerns

### Sybil Attacks and DHT Pollution

- **Issue**: In a permissionless DHT, generating a new identity is free. An
  attacker can generate millions of peerIDs and flood the DHT with
  announcements for a specific target embedding (or all embeddings), pushing
  legitimate announcements out of the Kademlia k-buckets.
- **Impact**: Denial of Service for discovery. The proposed mitigations (rate
  limiting, mutual signing, reputation decay) do not prevent a determined
  attacker from flooding the network with fresh, throwaway sybil nodes.
  "Stake-based signaling" (Lightning) fixes this but adds immense friction to
  onboarding.

### Privacy in Delegation

- **Issue**: Low-tier peers delegate computation (embeddings, ANN queries) to
  Supernodes. The request is encrypted with the Supernode's public key.
- **Impact**: The Supernode *must* decrypt the text to compute the embedding.
  This means the Supernode sees the user's raw text ("channel descriptions").
  This violates the "No servers. No surveillance" promise. A malicious
  Supernode can log all text and associate it with the requester's PeerID. The
  system relies entirely on a "no logging policy" honor system. ZK proofs are
  mentioned as "Future," but currently, delegation is a massive privacy hole.

### E2E Encryption Key Management

- **Issue**: The document states WebRTC DTLS secures all streams, which is
  correct. But it also mentions using libsodium `crypto_box_seal` for
  asynchronous/delegation messages, requiring conversion of Web Crypto
  `ed25519` signing keys to `x25519` encryption keys.
- **Impact**: Key conversion between Ed25519 and X25519 can lead to subtle
  cryptographic vulnerabilities if not implemented perfectly (e.g., using the
  correct base point and handling torsion components). Relying on this for core
  protocol security without explicit guardrails is a risk.

## Conclusion

While the concept of an ephemeral, semantic-routing-based P2P chat network is
innovative, the current ISC architecture relies on several contradictory
assumptions. Browser lifecycle constraints fundamentally clash with the
requirements of a stable DHT and persistent WebRTC routing. Furthermore, the
math underpinning the global LSH routing and supernode global views requires
significant revision to be practical at scale. Finally, the privacy guarantees
are broken the moment a user relies on a Supernode for delegation. Addressing
these issues will likely require moving away from a pure "browser-only" model
toward a hybrid model with more robust, persistent (and potentially
privacy-preserving) relay infrastructure.
