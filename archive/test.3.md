This is a comprehensive specification for ISC (Internet Semantic Chat). I'll create a test and simulation plan that builds a solid foundation and can grow into a full network simulation/optimization toolset.

## ISC Test & Simulation Plan

### Foundation Phase: Core Testing Infrastructure (Weeks 1-4)

#### 1. Unit Testing Framework

**Embedding & Semantic Core**
```javascript
// test/embedding.test.js
describe('Embedding Pipeline', () => {
  test('Tier detection accuracy', async () => {
    // Mock navigator APIs, verify tier selection
  });
  
  test('Model loading & caching', async () => {
    // Verify IndexedDB caching, tier-appropriate model selection
  });
  
  test('Cosine similarity correctness', () => {
    // Known vector pairs, edge cases (zero vectors, identical vectors)
  });
  
  test('Relational embedding composition', () => {
    // Test prompt construction, tag handling, weight application
  });
});
```

**Cryptographic Primitives**
```javascript
// test/crypto.test.js
describe('Identity & Signing', () => {
  test('Keypair generation deterministic with seed', () => {
    // ed25519 via Web Crypto API
  });
  
  test('Sign/verify roundtrip', async () => {
    // All payload types: announcements, posts, delegation requests
  });
  
  test('Signature validation rejects tampered data', async () => {
    // Mutation detection
  });
});
```

**LSH & DHT Key Generation**
```javascript
// test/lsh.test.js
describe('Locality-Sensitive Hashing', () => {
  test('Seeded RNG reproducibility', () => {
    // Same channel ID → same projection vectors
  });
  
  test('Collision properties', () => {
    // Similar vectors → similar hashes with high probability
  });
  
  test('LSH bucket distribution', () => {
    // Uniform distribution across hash space
  });
});
```

#### 2. Local Simulation Environment

**Deterministic P2P Simulation**
```javascript
// sim/libp2p-mock.js
class SimulatedLibp2p {
  constructor(config) {
    this.peers = new Map(); // In-memory peer registry
    this.dht = new SimulatedDHT();
    this.latencyModel = config.latencyModel || 'uniform';
  }
  
  // Simulate network partitions, churn, latency
  simulatePartition(peerIds) { }
  simulateChurn(ratePerMinute) { }
  setLatencyModel(model, params) { }
}
```

**Embedding Model Mock**
```javascript
// sim/embedding-mock.js
// Fast deterministic embeddings for simulation scale
class MockEmbeddingModel {
  constructor(dim = 384) {
    this.dim = dim;
    this.vocab = this.buildDeterministicVocab();
  }
  
  // O(1) embedding via hash-based projection
  // Preserves semantic relationships: similar text → similar vectors
  embed(text) {
    const hash = fnv1a(text);
    return this.projectToUnitSphere(hash, this.dim);
  }
  
  // Controlled similarity for testing matching logic
  embedWithTargetSimilarity(text, targetVector, similarity) {
    // Blend random + target to achieve exact cosine similarity
  }
}
```

### Growth Phase: Integration & Network Simulation (Weeks 5-12)

#### 3. Scenario-Based Testing

**Semantic Matching Scenarios**

| Scenario | Description | Success Criteria |
|----------|-------------|------------------|
| **Exact Match** | Two peers with identical descriptions | Similarity > 0.95, discovery < 5s |
| **Synonym Match** | "AI ethics" vs "machine morality" | Similarity > 0.75, discovery < 10s |
| **Relational Context** | Same topic, different locations | Location relation reduces match score appropriately |
| **Cross-Channel Drift** | Peer switches channels mid-chat | Graceful degradation, re-discovery |
| **Scale Test** | 1000 peers, 5 channels each | <100ms ANN query, <5% false negatives |

**Network Resilience Scenarios**

```javascript
// sim/scenarios/network-churn.js
async function testNetworkChurn() {
  const sim = new NetworkSimulation({
    peerCount: 100,
    churnRate: 0.1, // 10% leave per minute
    joinRate: 0.12  // 12% join per minute (net growth)
  });
  
  // Metrics: match availability, reconnection time, DHT consistency
  await sim.runFor(minutes: 10);
  
  assert(sim.metrics.matchAvailability > 0.95);
  assert(sim.metrics.medianReconnectionTime < 3000);
}
```

**Delegation Stress Testing**

```javascript
// sim/scenarios/delegation-load.js
async function testDelegationLoad() {
  const supernode = await createSupernode({ tier: 'high' });
  const clients = await createClients({ 
    count: 50, 
    tier: 'low',
    delegationEnabled: true 
  });
  
  // Ramp up request rate
  for (const rps of [1, 5, 10, 20, 50]) {
    await sim.loadTest({ requestsPerSecond: rps, duration: 60 });
    // Verify: latency < 500ms, success rate > 98%, no memory leaks
  }
}
```

#### 4. Visualization & Debugging Tools

**Real-Time Simulation Dashboard**
```javascript
// sim/dashboard/components/NetworkGraph.jsx
// D3.js force-directed graph of peers in embedding space
// - Color by channel
// - Edge thickness by similarity
// - Pulse animation for active DHT announcements
```

**Embedding Space Visualizer**
```javascript
// sim/dashboard/components/EmbeddingSpace.jsx
// Three.js 3D visualization (PCA-reduced from 384D)
// - Show clusters forming around topics
// - Highlight match candidates vs actual connections
// - Display LSH buckets as bounding boxes
```

**Metrics Pipeline**
```javascript
// sim/metrics/collector.js
class MetricsCollector {
  track(event, data) {
    // Structured logging for analysis
  }
  
  // Key metrics:
  // - Time-to-first-match distribution
  // - Semantic similarity accuracy (user-reported relevance)
  // - DHT query latency percentiles
  // - WebRTC connection success rate by NAT type
  // - Delegation request latency/success
  // - Model version distribution across network
}
```

### Optimization Phase: Performance & Tuning (Weeks 13-20)

#### 5. Parameter Optimization Framework

**Automated Tuning via Simulation**
```javascript
// sim/optimization/tuner.js
class ParameterTuner {
  // Bayesian optimization over simulation parameters
  
  async optimize({ 
    parameterSpace, // e.g., { lshHashes: [8, 12, 16, 20], refreshInterval: [60, 300, 600] }
    objective,      // e.g., minimize(matchLatency) subject to matchRate > 0.9
    budget          // simulation runs
  }) {
    // Use simulated annealing or Gaussian process optimization
  }
}
```

**A/B Testing Framework for Algorithms**
```javascript
// sim/experiments/matching-algorithms.js
async function compareMatchingAlgorithms() {
  const variants = [
    { name: 'baseline', ann: 'hnsw', lsh: 12 },
    { name: 'aggressive_lsh', ann: 'hnsw', lsh: 20 },
    { name: 'linear_fallback', ann: 'linear', lsh: 8 }
  ];
  
  for (const variant of variants) {
    const results = await runSimulation({
      ...variant,
      duration: 3600,
      peers: 500
    });
    report.push({ variant, metrics: results });
  }
}
```

#### 6. Chaos Engineering Suite

**Failure Injection**
```javascript
// sim/chaos/failures.js
const failureModes = {
  'dht_partition': () => /* isolate DHT buckets */,
  'model_version_split': () => /* 50% old model, 50% new */,
  'supernode_cascade': () => /* kill 80% of supernodes at once */,
  'browser_crash': (peer) => /* simulate tab close, no graceful exit */,
  'clock_skew': (peer) => /* offset timestamp by random seconds */,
  'sybil_attack': () => /* spawn 1000 malicious peers */
};
```

**Invariant Monitoring**
```javascript
// sim/chaos/invariants.js
const invariants = [
  // Safety
  (state) => state.peers.every(p => p.signatureValid),
  (state) => state.delegationQueue.length < 1000,
  
  // Liveness
  (state) => state.matchablePeers.length > 0.8 * state.totalPeers,
  (state) => state.medianLatency < 5000,
  
  // Consistency
  (state) => state.dhtEntries.every(e => e.ttl > 0),
  (state) => state.modelVersionEntropy < 2 // mostly homogeneous
];
```

### Full Network Simulation Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    ISC Simulation Orchestrator                   │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐ │
│  │  Scenario   │  │   Network   │  │    Metrics & Analysis   │ │
│  │   Engine    │  │   Topology  │  │                         │ │
│  │             │  │             │  │  - Latency distributions  │ │
│  │ • Churn     │  │ • Latency   │  │  - Match quality scores │ │
│  │ • Growth    │  │ • Bandwidth │  │  - Resource utilization   │ │
│  │ • Attacks   │  │ • NAT types │  │  - Convergence times      │ │
│  └──────┬──────┘  └──────┬──────┘  └─────────────────────────┘ │
│         │                │                                      │
│         └────────────────┼──────────────────────────────────────┘
│                          ▼                                      │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │              Simulated Peer Instances (1-10k)                │ │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐        ┌─────────┐       │ │
│  │  │ Peer #1 │ │ Peer #2 │ │ Peer #3 │  ...   │ Peer #N │       │ │
│  │  │ • Tier  │ │ • Tier  │ │ • Tier  │        │ • Tier  │       │ │
│  │  │ • DHT   │ │ • DHT   │ │ • DHT   │        │ • DHT   │       │ │
│  │  │ • ANN   │ │ • ANN   │ │ • ANN   │        │ • ANN   │       │ │
│  │  │ • RTC   │ │ • RTC   │ │ • RTC   │        │ • RTC   │       │ │
│  │  └─────────┘ └─────────┘ └─────────┘        └─────────┘       │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                          │                                      │
│                          ▼                                      │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                  In-Memory P2P Network Layer                  │ │
│  │     (Simulated libp2p with configurable latency/partitions)   │ │
│  └─────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

### Implementation Roadmap

| Phase | Deliverable | Timeline | Key Tools |
|-------|-------------|----------|-----------|
| **Foundation** | Unit test suite + mock embedding model | Week 1-2 | Jest, Vitest |
| | Deterministic LSH/DHT simulation | Week 3-4 | Custom WASM mocks |
| **Integration** | Multi-peer scenario runner | Week 5-6 | Node.js worker threads |
| | Visualization dashboard | Week 7-8 | D3.js, Three.js |
| | Delegation protocol testing | Week 9-10 | Custom crypto mocks |
| | Network resilience harness | Week 11-12 | Chaos monkey tools |
| **Optimization** | Parameter auto-tuner | Week 13-14 | Bayesian optimization lib |
| | A/B testing framework | Week 15-16 | Statistical analysis |
| | Performance regression suite | Week 17-18 | CI integration |
| **Scale** | 10k-peer simulation | Week 19-20 | Distributed simulation |

### Critical Test Scenarios for Launch

**Must Pass (Phase 1 - Trusted Network)**
1. Two browsers, same channel, find each other in <30s
2. Supernode assists low-tier client, verified result in <500ms
3. Channel edit → new matches within 10s
4. 50 concurrent peers, <5% connection failure
5. Graceful degradation when supernode disappears

**Must Pass (Phase 2 - Public Network)**
6. Sybil attack simulation: reputation system limits impact
7. Model version migration: no network split during 90-day window
8. 1000 peers, DHT query latency P95 <200ms
9. Delegation health metric accuracy >95%

This plan provides immediate value for validating the core ISC hypothesis—semantic proximity matching—while building toward sophisticated network optimization tools as the protocol matures.

