# ISC — Test & Simulation Plan

> **Purpose**: Build a solid testing foundation quickly, then grow into a full network simulation and optimization toolset across all four roadmap phases.

---

## Guiding Principles

- **Test the physics first** — the semantic matching math is the heart of ISC. Any bug here poisons everything downstream.
- **Simulate before you scale** — run a thousand virtual peers in-process before you run ten real browsers.
- **Degrade gracefully and verify it** — every tier, every fallback path, every failure mode must have a test.
- **Determinism is a first-class requirement** — seeded RNGs, fixed model weights, and reproducible DHT state make failures debuggable.

---

## Overview: Test Layers

```
┌────────────────────────────────────────────────────────┐
│  Layer 5 — Network Simulation (virtual peer swarms)    │
├────────────────────────────────────────────────────────┤
│  Layer 4 — Integration (multi-component flows)         │
├────────────────────────────────────────────────────────┤
│  Layer 3 — Protocol (DHT, WebRTC, delegation messages) │
├────────────────────────────────────────────────────────┤
│  Layer 2 — Component (embedding, LSH, ANN, crypto)     │
├────────────────────────────────────────────────────────┤
│  Layer 1 — Unit (pure functions: cosine, LSH, scoring) │
└────────────────────────────────────────────────────────┘
```

Layers 1–2 ship in Phase 1. Layers 3–5 grow progressively through Phases 2–4.

---

## Phase 1 — Foundation (Q1–Q2 2026)

### 1.1 Unit Tests — Pure Math & Utilities

These are zero-dependency, run instantly, and form the immune system for the matching core.

#### Cosine Similarity

| Test | Input | Expected |
|------|-------|----------|
| Identical vectors | `[1,0,0]`, `[1,0,0]` | `1.0` |
| Orthogonal vectors | `[1,0,0]`, `[0,1,0]` | `0.0` |
| Opposite vectors | `[1,0,0]`, `[-1,0,0]` | `-1.0` |
| Near-zero vector | `[0,0,0.00001]`, `[1,0,0]` | graceful NaN/0 (no divide-by-zero crash) |
| 384-dim random unit vectors | seeded random pairs | score in `[-1, 1]` always |
| Batch consistency | `sim(a,b) === sim(b,a)` | symmetric |

#### LSH Hashing

| Test | Scenario | Expected |
|------|----------|----------|
| Determinism | Same `(vec, channelId)` called twice | Identical hash string |
| Channel isolation | Same vec, different `channelId` | Different hash (seeded RNG diverges) |
| Bucket proximity | Two similar vectors (cosine > 0.9) | Collision rate ≥ 0.7 across 20 hashes |
| Dissimilar vectors | Cosine ≈ 0.1 | Collision rate ≤ 0.2 across 20 hashes |
| Hash length | Any input | Output exactly 32 chars per hash |
| Relation prefix | `"in_location:<hash>"` | Prefix correctly prepended |

#### Relational Matching Scorer

| Test | Scenario | Expected |
|------|----------|----------|
| Root-only alignment | No fused dists on either side | Returns root cosine score |
| Tag-match bonus | Matching `in_location` tags | Score ≥ equivalent score without tag bonus |
| Tag-mismatch | Different tags, similar vectors | Score = no bonus path |
| Weight scaling | Relation weight `2.0` vs `1.0` | Weighted score proportionally higher |
| Empty peer dists | Peer announces root only | Falls back to root-only comparison cleanly |
| Score normalization | Any valid input | Score always in `[0, 1]` |
| Spatiotemporal overlap | Two overlapping lat/long windows | Spatiotemporal bonus applied |
| No spatiotemporal overlap | 10,000 km apart | No bonus, no crash |

#### Monte Carlo Sampling

| Test | Scenario | Expected |
|------|----------|----------|
| Reproducibility | Seeded samples from `(μ, σ=0.1)` | Identical output given same seed |
| Spread = 0 | σ = 0 | All samples = μ (point distribution) |
| Large spread | σ = 1.0 | Sample mean ≈ μ within tolerance after 100 draws |
| High tier (100 samples) vs Low tier (20 samples) | Same distributions | Both converge; High tier closer to true score |

#### Signature Verification

| Test | Scenario | Expected |
|------|----------|----------|
| Valid signature | Correctly signed payload | `verifySignature()` returns `true` |
| Tampered payload | Bit-flip in `vec[0]` | `verifySignature()` returns `false` |
| Missing signature field | No `signature` key | Returns `false`, no crash |
| Wrong key | Payload signed by peer A, verified against peer B's key | Returns `false` |
| Replay detection | Same `requestID` seen twice | Second verification returns `false` |

---

### 1.2 Component Tests — Subsystems in Isolation

Each component runs with mocked I/O (no real network, no real model inference).

#### Embedding Pipeline (Stub Model)

Use a deterministic stub: map text → SHA-256 → reshape into 384-dim unit vector. This captures the interface without model download.

| Test | Scenario | Expected |
|------|----------|----------|
| Single embed | Text string → embedding | Shape `[384]`, L2 norm ≈ 1.0 (±0.01) |
| Relational composition | `computeRelationalDistributions(channel)` with 3 relations | Returns array of 4 distributions: 1 root + 3 fused |
| Max relations enforced | Channel with 6 relations | Throws or silently caps at 5 |
| Missing model | Model load fails | Graceful fallback to word-hash mode |
| Worker isolation | Embed called from Web Worker | No crash, correct result posted back |
| Caching | Same text embedded twice | Second call hits cache (IndexedDB stub), no recompute |

#### Device Tier Detection

| Test | Scenario | Expected |
|------|----------|----------|
| High tier | `hardwareConcurrency=8, deviceMemory=8, connection='4g'` | `'high'` |
| Mid tier | `hardwareConcurrency=4, deviceMemory=4, connection='4g'` | `'high'` (boundary) |
| Low tier | `hardwareConcurrency=2, deviceMemory=2` | `'low'` |
| Minimal trigger | `connection='2g'` | `'minimal'` |
| Memory trigger | `deviceMemory < 1` | `'minimal'` |
| API unavailable | All navigator fields undefined | Defaults to `'low'` (conservative) |
| Manual override | User sets tier in Settings | Override persists across refresh |

#### Channel State Machine

| Test | Scenario | Expected |
|------|----------|----------|
| Create channel | `POST /channels` (or local API) | Saved to localStorage; `id` matches schema format `ch_*` |
| Activate channel | Set channel active | Announce loop starts; DHT put called |
| Deactivate channel | Set channel inactive | Announce loop stops; TTL expires naturally |
| Edit while active | Change description mid-announce | Recompute embedding; new DHT put within 5 s |
| Duplicate channel | Fork a channel | New `id`, same description/relations, independent announce loop |
| Archive channel | Archive an active channel | DHT entry withdrawn; channel persists in localStorage |
| 5-channel limit (UI) | Attempt to add 6th active channel | UI blocks; warning shown |

#### Rate Limiter

| Test | Scenario | Expected |
|------|----------|----------|
| Under limit | 4 DHT puts in 60 s | All pass |
| At limit | Exactly 5 DHT puts in 60 s | 5th passes; counter saturated |
| Over limit | 6th put in same minute | Blocked; logged |
| Window reset | 61 s after first put | Counter resets; next put passes |
| Supernode enforcement | Low-tier peer sends 11 requests/min to supernode | Supernode rejects excess |

---

### 1.3 Integration Tests — End-to-End Flows (Two-Peer Local)

These use real libp2p nodes on localhost with real (stub or tiny) models, coordinated by the test harness.

#### Happy Path: Match and Chat

```
Peer A (High tier)  ─── embed "distributed systems, consensus" ───▶
Peer B (High tier)  ─── embed "Byzantine fault tolerance, Paxos" ──▶
                    ─── Both announce to shared in-memory DHT ─────▶
                    ─── A queries → B appears in ranked list ──────▶
                    ─── A dials B via /isc/chat/1.0 ────────────────▶
                    ─── Message exchange, delivery confirmed ────────▶
```

| Assertion | Expected |
|-----------|----------|
| Cosine sim(A, B) | ≥ 0.70 threshold |
| B in A's candidate list | Within first refresh cycle (≤ 30 s) |
| Chat message round-trip latency | ≤ 2 s (localhost) |
| No self-match | A never sees its own peerID |

#### Dissimilar Peers (No Match)

Peer A: "ceramics, fermentation" vs. Peer B: "high-frequency trading algorithms"

| Assertion | Expected |
|-----------|----------|
| Cosine sim | ≤ 0.55 |
| B not in A's match list | Filtered below threshold |
| No unsolicited chat dial | Neither peer dials the other |

#### Model Mismatch Rejection

Peer A announces with `model: "Xenova/all-MiniLM-L6-v2"`, Peer B with a spoofed `model: "fake-model-v1"`.

| Assertion | Expected |
|-----------|----------|
| B's announcement | Silently discarded at query refinement |
| A's match list | Empty (or only same-model peers) |
| No crash | True |

#### Signature Rejection

Peer A announces a tampered payload (valid structure, invalid signature).

| Assertion | Expected |
|-----------|----------|
| Peer B discards A's announcement | True |
| No error propagation | No crash in querying peer |

#### Group Chat Formation

Three peers (A, B, C) all embed semantically similar text.

| Assertion | Expected |
|-----------|----------|
| Each peer sees the others in ranked list | True |
| Mesh formed | All three dialed into group |
| Centroid key announced to DHT | True |
| Latecomer peer D (similar embedding) | Finds centroid key, joins group |

---

### 1.4 Supernode Delegation Tests

#### Delegation Happy Path

Low-tier peer requests embedding from High-tier supernode.

| Step | Assertion |
|------|-----------|
| Low tier sends encrypted `delegate_request` | Request reaches supernode |
| Supernode returns signed `delegate_response` | Signature valid |
| Embedding norm | ≈ 1.0 (±0.01) |
| Model version in response | Matches canonical |
| Request ID matches | True |
| End-to-end latency | ≤ 5,000 ms |

#### Fallback Chain

| Scenario | Expected Behavior |
|----------|-------------------|
| No supernodes in DHT | Low tier falls back to `gte-tiny` locally |
| Supernode times out (5,001 ms) | Request aborted; local fallback triggered |
| Supernode returns invalid signature | Result discarded; local fallback triggered |
| Supernode returns wrong model hash | Result discarded; local fallback triggered |
| Supernode crashes mid-request | Reconnect attempted once; then local fallback |

#### Rate Limit Enforcement

| Scenario | Expected |
|----------|----------|
| 3 delegation requests/min | All served |
| 4th request within 60 s | Blocked at supernode; HTTP 429 equivalent |
| `maxConcurrent=5` hit | 6th concurrent request queued or rejected |

---

## Phase 2 — Scale & Safety (Q3–Q4 2026)

### 2.1 Network Simulation Framework

The simulation harness runs hundreds of virtual peers in a single Node.js process, sharing an in-memory DHT stub and deterministic embedding stubs.

#### Architecture

```javascript
// sim/harness.js
class NetworkSim {
  peers     = new Map();   // peerID → VirtualPeer
  dht       = new InMemoryDHT();
  clock     = new SimClock();  // virtual time; no real setTimeout

  addPeer(config)          // creates VirtualPeer with tier + description
  tick(ms)                 // advances clock; triggers announces, queries, decays
  injectChurn(rate)        // randomly disconnect peers
  injectPartition(peerIds) // isolate a subset
  getMatchGraph()          // returns adjacency matrix of all match scores
  getMetrics()             // returns P50/P95/P99 latency, connection rate, etc.
}
```

#### Baseline Simulation Scenarios

| Scenario | Peers | Duration | Success Criteria |
|----------|-------|----------|-----------------|
| Dense cluster | 50 peers, 5 semantic topics | 10 min virtual | All peers find ≥ 1 match in ≤ 10 s |
| Sparse distribution | 50 peers, 50 unique topics | 10 min virtual | No false matches above 0.75 threshold |
| Mixed tiers | 20 High + 20 Mid + 10 Low | 10 min virtual | Low-tier peers delegated successfully |
| Phase 1 success gate | 50+ concurrent | 30 min virtual | < 5% connection failure; < 10 s median first-match |

---

### 2.2 Relational Embedding Simulation

Test that compositional embeddings improve match precision over root-only matching.

#### Experiments

**Experiment A: Location Filtering Lift**

- 100 peers, all embedded "machine learning research"
- 50 in Tokyo (`in_location: lat:35.6895, long:139.6917`), 50 in Berlin
- Measure: % of Tokyo peers matched to other Tokyo peers vs. Berlin peers

Expected: Relational matching lifts within-city match rate by ≥ 20% vs root-only.

**Experiment B: Mood Separation**

- 40 peers, same topic "startup strategy"
- 20 with `with_mood: energetic and optimistic`, 20 with `with_mood: cautious and analytical`
- Measure: Intra-mood similarity vs. cross-mood similarity

Expected: Intra-mood score > cross-mood score on average.

**Experiment C: Tag-Match Bonus Calibration**

- Vary the 1.2× tag-match bonus from 1.0 to 2.0
- Measure: precision@5 for the match list
- Expected: Optimal bonus identified empirically; commit as default constant.

---

### 2.3 Reputation System Simulation

| Scenario | Setup | Expected |
|----------|-------|----------|
| Honest network | 100 peers, all normal | Reputation converges to uniform distribution |
| Sybil cluster | 10 sybil peers with fake mutual positive votes | Reputation gaming detected; sybil scores capped |
| Reputation decay | Peer goes offline for 30 simulated days | Score decays to baseline by half-life rule |
| Mute propagation | High-rep peer mutes spammer | Spammer deprioritized for peer's neighbors |

---

### 2.4 DHT Performance Benchmarks

| Benchmark | Metric | Target |
|-----------|--------|--------|
| Put throughput | DHT puts/sec (in-process) | ≥ 1,000 |
| Get latency (cold) | Time to first result | ≤ 50 ms |
| Get latency (warm) | Cached result | ≤ 5 ms |
| Stale entry cleanup | After TTL=300 s | Entries purged within 1 TTL cycle |
| Scale | 10,000 entries in DHT | Get still ≤ 100 ms |

---

### 2.5 Model Version Migration Simulation

Simulate the 90-day dual-announcement migration window.

| Phase | Scenario | Expected |
|-------|----------|----------|
| Pre-migration | All peers on v1 | Normal matching |
| Migration start | 10% adopt v2; dual-announce enabled on High tier | v2 peers match each other; v1 peers see v1 peers |
| 50% migration | Clients show migration prompt | Prompt triggered for v1 peers |
| Post-migration (day 91) | v1 TTLs expire | v1 peers isolated in compatibility shard; v2 network intact |

---

### 2.6 Security Simulation

#### Malicious Supernode

A supernode returns plausible-looking but incorrect embeddings (adversarial perturbation).

| Test | Expected |
|------|----------|
| Local sanity check: norm deviation > 0.01 | Result discarded |
| Cross-check via local minimal model (optional flag) | Discrepancy detected; supernode flagged |
| Consistent bad responses (> 3 in a row) | Supernode deprioritized in selection |

#### Sybil Flood

An attacker creates 1,000 sybil peers all announcing proximity to a target peer.

| Test | Expected |
|------|----------|
| Target's match list | Not flooded (candidate cap + dedup enforced) |
| Rate limiter | Sybil announcements throttled at ≤ 5 puts/min per peerID |
| Phase 1 (trusted network) | Social invite barrier prevents most sybils at entry |

#### Replay Attack

Attacker captures a valid `delegate_request` and replays it 60 s later.

| Test | Expected |
|------|----------|
| `timestamp` check | Request older than 30 s rejected |
| `requestID` dedup | Second use of same UUID rejected |

---

## Phase 3 — Social Layer (2027)

### 3.1 Feed Quality Metrics

| Metric | Definition | Target |
|--------|------------|--------|
| Semantic precision@10 | % of top-10 feed posts with sim ≥ 0.6 to active channel | ≥ 80% |
| Serendipity rate | % of feed posts from non-obvious semantic neighbors (sim 0.55–0.70) | 15–25% |
| Echo chamber score | Mean inter-post similarity within a user's feed | ≤ 0.85 (diversity maintained) |
| Chaos mode lift | Delta serendipity rate when chaos mode enabled | ≥ +10% |

### 3.2 Social Graph Simulation

| Scenario | Peers | Metric |
|----------|-------|--------|
| Organic follow growth | 500 peers, 30-day sim | Follow graph diameter ≤ 6 (small world) |
| Web of Trust convergence | Mutual-follow chains | Rep scores stabilize within 7 simulated days |
| Moderation event propagation | 1 report from high-rep peer | Propagates to ≥ 80% of network within 24 h sim |

### 3.3 Engagement Simulation

Validate that PubSub trending logic surfaces genuine high-engagement clusters.

| Test | Setup | Expected |
|------|-------|----------|
| Organic viral post | 30% of network engages with one post | Surfaces in Global Explore |
| Synthetic bot engagement | 50 sybil likes on a post | Does not surface (rep weighting discounts low-rep peers) |
| TTL expiry | Trending cluster from 48 h ago | No longer in Explore feed |

---

## Phase 4 — Ecosystem (2028+)

### 4.1 Interoperability Tests

| Test | Scope | Expected |
|------|-------|----------|
| AT Protocol bridge | ISC post → Bluesky DID-linked record | Record valid; resolvable via atproto |
| Vector import/export | Export channel distributions as JSON; re-import | Embeddings match within float precision |
| Cross-instance matching | Two ISC deployments (enterprise + public) | Semantic matching works if same model version |

### 4.2 Enterprise / Private Instance Tests

| Test | Scenario | Expected |
|------|----------|----------|
| Isolated DHT | Private instance has no public bootstrap peers | No leakage to public network |
| Cross-instance federation | Admin explicitly bridges two instances | Federated matching only for allowed channels |

---

## Continuous Optimization Loop

Once the simulation harness is in place (Phase 2), run the following as a scheduled optimization suite:

### Tunable Parameters

| Parameter | Current Default | Simulation Knob |
|-----------|----------------|-----------------|
| Match threshold | 0.75 | Sweep 0.60–0.90 in 0.05 steps |
| Tag-match bonus | 1.2× | Sweep 1.0–2.0 |
| LSH hash count | 20 (High) | Sweep 8–32 |
| Monte Carlo samples | 100 (High) | Sweep 20–200 |
| DHT candidate cap | 100 (High) | Sweep 20–200 |
| Refresh interval | 5 min (High) | Sweep 1–15 min |
| Supernode rate limit | 10 req/min | Sweep 5–30 |
| Spatiotemporal bonus weight | 0.5 | Sweep 0.1–1.0 |

### Optimization Objectives

For each parameter set, simulate 500-peer networks and record:

- **Median time-to-first-match** (minimize)
- **Connection failure rate** (minimize)
- **False-positive match rate** (dissimilar peers matched; minimize)
- **False-negative match rate** (similar peers missed; minimize)
- **DHT put bandwidth** (minimize)
- **Delegation success rate** (maximize)

Use a simple grid search first; add Bayesian optimization in Phase 3 when the search space grows.

---

## Test Infrastructure

### Tooling Recommendations

| Layer | Tool | Notes |
|-------|------|-------|
| Unit / component | Vitest | Fast, ESM-native; ideal for browser-targeted JS |
| Integration | Playwright | Two real browser tabs; real WebRTC |
| Simulation harness | Node.js + in-memory DHT stub | No browser needed; 1,000 virtual peers feasible |
| Coverage | c8 / Istanbul | Track which paths are exercised |
| Benchmarks | `tinybench` | Microbenchmarks for cosine, LSH, ANN |
| CI | GitHub Actions | Run units + component tests on every PR; simulations nightly |

### Directory Layout

```
tests/
├── unit/
│   ├── cosine.test.js
│   ├── lsh.test.js
│   ├── relational-match.test.js
│   ├── monte-carlo.test.js
│   └── signature.test.js
├── component/
│   ├── embedding.test.js
│   ├── tier-detection.test.js
│   ├── channel-state.test.js
│   └── rate-limiter.test.js
├── integration/
│   ├── two-peer-match.test.js
│   ├── group-chat.test.js
│   ├── delegation.test.js
│   └── model-mismatch.test.js
├── simulation/
│   ├── harness/
│   │   ├── NetworkSim.js
│   │   ├── VirtualPeer.js
│   │   ├── InMemoryDHT.js
│   │   └── SimClock.js
│   ├── scenarios/
│   │   ├── dense-cluster.sim.js
│   │   ├── mixed-tiers.sim.js
│   │   ├── sybil-flood.sim.js
│   │   └── model-migration.sim.js
│   └── optimization/
│       └── parameter-sweep.js
└── fixtures/
    ├── embeddings/          # pre-computed stub vectors
    ├── channels/            # example channel JSON
    └── keypairs/            # test ed25519 keypairs (never reuse in prod)
```

### CI Pipeline Gates

| Gate | Runs On | Required to Merge |
|------|---------|------------------|
| Unit tests (all pass) | Every PR | ✅ Yes |
| Component tests (all pass) | Every PR | ✅ Yes |
| Integration tests (happy path) | Every PR | ✅ Yes |
| Integration tests (all paths) | Nightly | ⚠️ Blocks release |
| Simulation baseline (50 peers) | Nightly | ⚠️ Blocks release |
| Security checklist (manual) | Pre-merge for crypto/delegation PRs | ✅ Yes |
| Parameter sweep optimization | Weekly | ℹ️ Informational |

---

## Phase 1 Launch Readiness Checklist

Before trusted-network launch, all of the following must pass:

- [ ] All unit tests green
- [ ] All component tests green
- [ ] Two-peer integration: happy path, model mismatch, signature rejection
- [ ] Supernode delegation: happy path, all five fallback scenarios
- [ ] Rate limiter: over-limit blocked, window resets correctly
- [ ] 50-peer simulation: < 5% connection failure; < 10 s median first-match
- [ ] Security checklist (from README) fully verified
- [ ] Tier detection tested on at least: Chrome desktop, Chrome Android, Safari iOS, Firefox desktop
- [ ] No private keys logged anywhere in DevTools (manual audit)

---

*This plan is a living document. Each phase should add new scenarios based on what the previous phase reveals. The simulation harness is the most leveraged investment — build it right in Phase 2 and it pays dividends through Phase 4.*

