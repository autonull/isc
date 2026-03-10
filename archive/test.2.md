Based on the **ISC — Internet Semantic Chat** specification, here is a comprehensive **Test and Simulation Plan**. This plan is designed to validate the core decentralized architecture quickly (Phase 1) while evolving into a robust network simulation toolkit for long-term optimization (Phase 4).

---

# ISC Test & Simulation Plan
**Objective:** Validate semantic matching reliability, P2P connectivity, and supernode delegation security while building a scalable network simulation environment.

## Phase 1: Core Component Validation (The Foundation)
**Goal:** Ensure individual building blocks work correctly before network integration. Focus on deterministic behavior in a browser environment.
**Timeline:** Weeks 1–4 (Aligns with Roadmap Phase 1 MVP)

### 1.1 Embedding & Model Consistency
*Critical Risk:* "Mixing models breaks the network."
*   **Test:** **Cross-Model Vector Compatibility**
    *   Verify that `all-MiniLM-L6-v2` produces identical vectors for identical inputs across different browser sessions.
    *   Verify that `model_version` hashes match exactly before cosine similarity calculation.
    *   **Simulation:** Run 100 identical prompts on 5 different browser profiles; assert vector cosine similarity == 1.0.
*   **Test:** **Tier Fallback Logic**
    *   Assert `Low` tier loads `gte-tiny` and `Minimal` tier loads Word-hash fallback.
    *   Assert `computeRelationalDistributions` skips relations on `Low` tier.

### 1.2 Cryptographic Integrity
*Critical Risk:* Identity spoofing and tampered announcements.
*   **Test:** **Key Pair Generation & Persistence**
    *   Verify `ed25519` keypair generation via Web Crypto API.
    *   Verify keys persist in IndexedDB across refreshes but are unique per installation.
*   **Test:** **Signature Verification**
    *   Create a unit test suite for `verifySignature(peer)`.
    *   Inject tampered DHT payloads (modified vector, modified TTL) and assert rejection.

### 1.3 LSH & DHT Mapping
*Critical Risk:* Bucket collision or poor distribution.
*   **Test:** **LSH Determinism**
    *   Assert that `lshHash(vec, channelId, seed)` produces identical keys for identical inputs.
    *   **Simulation:** Generate 10,000 random vectors; plot distribution across LSH buckets to ensure uniformity (avoid hotspots).
*   **Test:** **TTL Expiry**
    *   Assert DHT entries expire correctly after `ttl` seconds.
    *   Simulate clock skew to ensure clients discard stale `updatedAt` timestamps.

---

## Phase 2: Localized Network Simulation (The Lab)
**Goal:** Simulate a multi-peer network in a controlled environment to test discovery and chat formation.
**Timeline:** Weeks 5–8 (Aligns with Roadmap Phase 1 Core Reliability)

### 2.1 Multi-Peer Browser Cluster
*   **Tooling:** Use **Playwright** or **Puppeteer** to spin up multiple headless browser instances acting as peers.
*   **Test:** **Peer Discovery Latency**
    *   Spawn 50 peers with similar channel descriptions.
    *   Measure time from `Embed` click to `Match Found` (Target: <10s median).
*   **Test:** **WebRTC Connection Success Rate**
    *   Attempt 1,000 dial attempts between peers.
    *   Track failure modes (NAT traversal, ICE candidate failure).
    *   **Metric:** Target >95% connection success in LAN/localhost sim.

### 2.2 Semantic Matching Accuracy
*   **Test:** **Proximity Ranking**
    *   Create ground-truth datasets (e.g., 10 peers talking about "AI", 10 about "Cooking").
    *   Assert that "AI" peers rank higher in each other's match lists than "Cooking" peers.
    *   **Metric:** Precision@10 (How many of the top 10 matches are semantically relevant?).
*   **Test:** **Relational Binding**
    *   Peer A: "AI ethics **in_location** Tokyo"
    *   Peer B: "AI ethics **in_location** New York"
    *   Peer C: "AI ethics **in_location** Tokyo"
    *   Assert Peer A matches Peer C higher than Peer B due to relational fusion.

### 2.3 Channel Dynamics
*   **Test:** **Thought Drift**
    *   Simulate a peer editing their channel description.
    *   Verify DHT announcement updates within `TIER.refreshInterval`.
    *   Verify old matches decay and new matches appear.
*   **Test:** **Multi-Channel Independence**
    *   Peer maintains "Work" and "Evening" channels.
    *   Assert chats from "Work" do not bleed into "Evening" match lists.

---

## Phase 3: Stress & Dynamics Testing (The Field)
**Goal:** Test scalability, supernode delegation, and adversarial conditions.
**Timeline:** Weeks 9–12 (Aligns with Roadmap Phase 2 Scale & Safety)

### 3.1 Supernode Delegation Load
*   **Simulation:** **High/Low Tier Mix**
    *   Spawn 10 `High` tier peers (Supernodes) and 90 `Low` tier peers.
    *   Low peers request embedding assistance.
    *   **Metric:** Measure `avgLatencyMs` on `delegation_health`. Target <500ms.
    *   **Metric:** Measure Supernode CPU/Memory usage in browser context.
*   **Test:** **Verification Overhead**
    *   Measure time taken for Low peer to verify Supernode signature + embedding norm.
    *   Assert verification does not block UI thread (must run in Web Worker).

### 3.2 Network Churn & Stability
*   **Simulation:** **Peer Churn**
    *   Randomly kill 20% of peers every 60 seconds.
    *   Assert remaining peers re-discover matches via DHT refresh.
    *   Assert chat streams handle reconnection logic (5s wait -> redial).
*   **Simulation:** **Cold Start**
    *   Start network with 0 peers, add 100 over 5 minutes.
    *   Measure time until first successful match for early adopters.

### 3.3 Threat Model Simulation
*   **Test:** **Malicious Supernode**
    *   Spawn a supernode returning random/incorrect embeddings.
    *   Assert Low-tier peers reject results via `Local Verification` (norm check + cross-check subset).
*   **Test:** **Sybil Attack**
    *   Spawn 500 peers with identical keys (simulated compromise) or similar vectors.
    *   Verify Rate Limiting (max 5 DHT puts/minute) throttles the attack.
    *   Verify Reputation weighting (Phase 2) deprioritizes new/low-rep peers.

---

## Phase 4: The ISC Simulation Toolkit (The Product)
**Goal:** Formalize the testing infrastructure into a reusable optimization tool set for future development and community research.
**Timeline:** Q3 2026+ (Aligns with Roadmap Phase 2/3)

### 4.1 ISC-NET Simulator Architecture
Build a standalone Node.js/WASM package that allows developers to script network scenarios without launching full browsers.
*   **Virtual Peer Class:** Abstracts libp2p, Embedding, and DHT logic into a lightweight JS object.
*   **Network Condition Injector:** Simulate 2G/3G/4G latency, packet loss, and NAT types.
*   **Behavioral Scripts:** Define peer behaviors (e.g., "Chatter", "Lurker", "Supernode", "Attacker").

### 4.2 Optimization Dashboards
*   **Vector Space Visualizer:** 2D/3D projection (t-SNE/UMAP) of active peer distributions to visualize clustering.
*   **DHT Heatmap:** Visualize key distribution to identify LSH bucket imbalances.
*   **Delegation Economy Tracker:** Track request/response volumes to tune `rateLimit` and `maxConcurrent` settings.

### 4.3 Continuous Integration (CI) Pipeline
*   **Embedding Regression Test:** Run on every PR to ensure model updates don't shift vector space unexpectedly.
*   **Security Audit Suite:** Automated fuzzing of DHT payload parsers and signature verifiers.
*   **Performance Budget:** Fail build if `High` tier model load time exceeds 2s or memory usage exceeds 500MB.

---

## Key Metrics & KPIs
| Category | Metric | Target (Phase 1) | Target (Phase 2) |
| :--- | :--- | :--- | :--- |
| **Matching** | Time-to-First-Match | <10s | <5s |
| **Matching** | Precision@10 (Semantic) | >0.7 | >0.85 |
| **Network** | Connection Success Rate | >90% | >95% |
| **Delegation** | Avg Response Latency | <1000ms | <500ms |
| **Delegation** | Verification Failure Rate | 0% | <1% (false positives) |
| **Resource** | Model Load Time (High Tier) | <3s | <2s |
| **Resource** | Memory Footprint (Idle) | <200MB | <150MB |
| **Safety** | Spam Announcement Rate | <5/hr | <1/hr (with rep) |

---

## Recommended Tooling Stack
| Component | Tool | Reason |
| :--- | :--- | :--- |
| **Browser Automation** | **Playwright** | Supports multi-context (incognito) for peer simulation. |
| **Network Sim** | **Toxiproxy** | Inject latency/bandwidth limits between peer instances. |
| **Unit Testing** | **Vitest** | Fast, browser-compatible test runner for WASM/JS logic. |
| **E2E Testing** | **Custom Harness** | Node.js script spawning multiple `js-libp2p` nodes without UI. |
| **Visualization** | **Deck.gl / D3** | For Vector Space and DHT Heatmap dashboards. |
| **CI/CD** | **GitHub Actions** | Run headless browser tests on every commit. |

## Immediate Next Steps (Week 1)
1.  **Initialize Vitest Suite:** Set up unit tests for `computeRelationalDistributions` and `lshHash`.
2.  **Build "Peer-in-a-Box":** Create a Node.js script that instantiates a libp2p node + embedding model without the UI, allowing rapid scripting of peer interactions.
3.  **Model Hash Locking:** Implement the `model_version` check in the DHT announcement logic to prevent accidental cross-model matching during development.
4.  **Security Baseline:** Run a static analysis scan on the Web Crypto API usage to ensure key storage is secure in IndexedDB.

This plan ensures you validate the critical "semantic geometry" and "decentralized trust" assumptions early, while building the infrastructure needed to scale the network safely in later phases.

