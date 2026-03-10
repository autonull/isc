# ISC Documentation Remediation Plan

> **Purpose**: Fix all critical issues identified in documentation analysis and verify against original architecture.
>
> **Timeline**: 4-6 weeks documentation sprint before implementation begins.

---

## Phase 0: Verification Against Original (Week 1)

### 0.1 Content Recovery Check

Verify all content from original files was preserved during sharding:

- [ ] Compare `archive/architecture.original.md` (648 lines) → `CODE.md` + `README.md`
- [ ] Compare `archive/monorepo.original.md` (660 lines) → `CODE.md`
- [ ] Compare `archive/semantic.md` (179 lines) → `SEMANTIC.md`
- [ ] Compare `archive/social.md` (71 lines) → `SOCIAL.md`
- [ ] Compare `archive/safety.md` (74 lines) → `SECURITY.md`
- [ ] Compare `archive/channels.md` (42 lines) → `PROTOCOL.md`
- [ ] Compare `archive/embeddings.md` (54 lines) → `SEMANTIC.md`

**Deliverable**: Content mapping spreadsheet showing every section from originals and where it landed in sharded docs.

### 0.2 Lost Content Recovery

Identify and recover any content lost during sharding:

- [ ] Original dependency graph (architecture.original.md:401-420) → Add to CODE.md
- [ ] Workspace configuration details (architecture.original.md:422-470) → Add to CODE.md
- [ ] Code sharing matrix (architecture.original.md:472-480) → Add to CODE.md
- [ ] Migration strategy (architecture.original.md:482-510) → Add to CODE.md
- [ ] Future form factors table (architecture.original.md:620-630) → Add to CODE.md
- [ ] Example implementations (architecture.original.md:632-648) → Add to CODE.md

**Deliverable**: CODE.md restored to full original content.

---

## Phase 1: Critical Technical Fixes (Weeks 1-2)

### 1.1 LSH Implementation — CRITICAL BLOCKER

**File**: `PROTOCOL.md` (lines 103-117)

**Current (BROKEN)**:
```javascript
function lshHash(vec: number[], channelId: string, numHashes: number = 20, hashLen: number = 32): string[] {
  const rng = seededRng(channelId);
  const hashes: string[] = [];
  for (let i = 0; i < numHashes; i++) {
    const proj = vec.map(() => rng() * 2 - 1);  // BUG: New random per element!
    const bits = vec.map((v, j) => (v * proj[j] > 0 ? '1' : '0')).join('');
    hashes.push(bits.slice(0, hashLen));
  }
  return hashes;
}
```

**Task**:
- [ ] Research correct random projection LSH implementation
- [ ] Fix: Generate ONE projection vector per hash, not per element
- [ ] Add unit test specification for locality-sensitivity property
- [ ] Add verification: similar vectors should produce similar hashes

**Corrected Implementation** (to be added):
```javascript
function lshHash(vec: number[], channelId: string, numHashes: number = 20, hashLen: number = 32): string[] {
  const rng = seededRng(channelId);
  const hashes: string[] = [];
  
  for (let i = 0; i < numHashes; i++) {
    // Generate ONE projection vector (consistent across all elements)
    const proj = Array.from({ length: vec.length }, () => rng() * 2 - 1);
    
    // Project vector onto random hyperplane
    const bits = proj
      .map((p, j) => (vec[j] * p > 0 ? '1' : '0'))
      .join('');
    
    hashes.push(bits.slice(0, hashLen));
  }
  return hashes;
}
```

**Verification**: Test with known similar/dissimilar vector pairs.

---

### 1.2 Spatiotemporal Similarity — MISSING IMPLEMENTATION

**File**: `SEMANTIC.md` (lines 103-119)

**Current**: Calls undefined `haversineDistance` function.

**Task**:
- [ ] Add `haversineDistance` implementation
- [ ] Add `parseLocation` complete implementation
- [ ] Add `parseTime` complete implementation
- [ ] Add `locationOverlap` complete implementation
- [ ] Add `timeOverlap` complete implementation
- [ ] Add unit test specifications

**Implementation** (to be added):
```javascript
function haversineDistance(lat1: number, lon1: number, lat2: number, lon2: number): number {
  const R = 6371; // Earth's radius in km
  const dLat = (lat2 - lat1) * Math.PI / 180;
  const dLon = (lon2 - lon1) * Math.PI / 180;
  const a = Math.sin(dLat/2) * Math.sin(dLat/2) +
            Math.cos(lat1 * Math.PI / 180) * Math.cos(lat2 * Math.PI / 180) *
            Math.sin(dLon/2) * Math.sin(dLon/2);
  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
  return R * c;
}

function locationOverlap(a: Location, b: Location): number {
  const distance = haversineDistance(a.lat, a.long, b.lat, b.long);
  const maxRadius = Math.max(a.radius, b.radius);
  return Math.max(0, 1 - (distance / (maxRadius * 2)));
}

function timeOverlap(a: TimeWindow, b: TimeWindow): number {
  const overlap = Math.max(0, Math.min(a.end.getTime(), b.end.getTime()) 
                         - Math.max(a.start.getTime(), b.start.getTime()));
  const total = Math.max(a.end.getTime(), b.end.getTime()) 
              - Math.min(a.start.getTime(), b.start.getTime());
  return total > 0 ? overlap / total : 0;
}
```

---

### 1.3 Crypto Implementation — RESOLVE CONTRADICTION

**Files**: `README.md` vs `DELEGATION.md`

**Issue**: README specifies "Web Crypto API (ed25519)" but DELEGATION.md uses `libsodium-wrappers`.

**Task**:
- [ ] Decide: Web Crypto API only, or libsodium for encryption?
- [ ] Update all documents to consistent crypto stack
- [ ] If Web Crypto API: specify how to handle encryption (not just signing)
- [ ] If libsodium: update README tech stack, add dependency spec
- [ ] Specify key format (raw, PKCS#8, SPKI)
- [ ] Specify signature format (raw bytes, base64, etc.)

**Recommended Resolution**:
- Use Web Crypto API for ed25519 signing/verification (native browser support)
- Use libsodium-wrappers for sealed-box encryption (not available in Web Crypto)
- Document this split explicitly in all files

---

### 1.4 Cold Start Mechanism — ADD SPECIFICATION

**Files**: All (missing from all)

**Task**:
- [ ] Specify first-user experience (empty DHT scenario)
- [ ] Define bootstrap peer discovery beyond hardcoded list
- [ ] Add "seed tab" pattern specification
- [ ] Specify minimum viable network size
- [ ] Add QR code / invite link mechanism for direct peer connection
- [ ] Specify DNS-based peer discovery as DHT fallback

**Specification** (to be added to PROTOCOL.md):
```markdown
### Cold Start Protocol

1. **Empty DHT Handling**:
   - If DHT query returns 0 results after 3 attempts, trigger cold start
   - Display message: "No peers online. Waiting for connections..."
   - Continue announcing own presence every 30 seconds

2. **Bootstrap Peer Discovery**:
   - Primary: Hardcoded bootstrap peers (see Bootstrap Peers section)
   - Secondary: DNS-based discovery (`_isc._tcp.bootstrap.isc.network`)
   - Tertiary: Community-maintained peer list (HTTPS endpoint)

3. **Seed Tab Pattern**:
   - User can open "seed tab" that remains open
   - Seed tab acts as local bootstrap peer for other tabs
   - Uses WebRTC + libp2p in-browser relay

4. **Direct Connection Mechanisms**:
   - QR code containing peer ID + signaling URL
   - Invite link: `https://isc.network/?peer=<peerID>&relay=<relayURL>`
   - Manual peer ID entry with "Dial" button
```

---

### 1.5 Group Chat Formation — SPECIFY THRESHOLDS

**Files**: `README.md`, `PROTOCOL.md`

**Issue**: "Dense clusters" and "3+ peers" undefined.

**Task**:
- [ ] Define density threshold (peers per vector radius)
- [ ] Specify maximum group size
- [ ] Define centroid computation algorithm
- [ ] Specify room ID generation
- [ ] Add mesh formation protocol

**Specification** (to be added to PROTOCOL.md):
```markdown
### Group Chat Formation

**Density Threshold**: 3+ peers within 0.15 cosine similarity of centroid

**Maximum Group Size**: 8 peers (WebRTC connection limit consideration)

**Formation Protocol**:
1. When peer detects 3+ matches within 0.15 similarity:
   - Compute centroid: `mean([vec1, vec2, ...])`
   - Generate room ID: `sha256(centroid + channelID + timestamp_hour)`
   - Announce room membership to DHT at `/isc/group/<roomID>`

2. Mesh Formation:
   - First peer (by peerID sort order) becomes coordinator
   - Coordinator dials all members
   - Members accept and form full mesh

3. Group Maintenance:
   - Heartbeat every 30 seconds
   - If member drops below 0.55 similarity, graceful exit prompt
   - If coordinator leaves, new coordinator elected (lowest peerID)
```

---

## Phase 2: Resolve Inconsistencies (Week 2)

### 2.1 Timeline Contradiction — SOCIAL.md vs ROADMAP.md

**Issue**: SOCIAL.md states Q2-Q3 2026 for social features; ROADMAP.md states Q1-Q2 2027.

**Task**:
- [ ] Decide actual timeline (9-month discrepancy)
- [ ] Update SOCIAL.md "Indicative timeline" line
- [ ] Update ROADMAP.md Phase 3 section
- [ ] Ensure consistency across all references

**Recommended Resolution**: ROADMAP.md is authoritative (Q1-Q2 2027). Update SOCIAL.md.

---

### 2.2 Parameter Consistency Table

**Task**: Create single source of truth for all parameters:

| Parameter | Current Values | Resolution |
|-----------|---------------|------------|
| **Announcement TTL** | "5-min" (README) vs tier-dependent (PROTOCOL) | Use tier-dependent; update README |
| **Similarity Threshold** | "~0.7" vs "0.55-0.85" vs "min 0.55" | Use ranges from PROTOCOL.md; update others |
| **Model Sizes** | "~4-22 MB" vs "22,18,8,4 MB" | Use detailed table from SEMANTIC.md |
| **Deployment Modes** | 3 modes (README) vs 2 modes (SECURITY) | Use 3 modes; update SECURITY.md |
| **UI Technology** | "Vanilla HTML/JS" | Keep; add "no framework dependencies" |

**Files to Update**:
- [ ] README.md (lines 189-193, 279, 292)
- [ ] SECURITY.md (lines 72, 84)
- [ ] All other references

---

### 2.3 Post Schema Circular Reference

**File**: `SOCIAL.md` (lines 24-52)

**Issue**: Can't sign object that includes signature field.

**Task**:
- [ ] Fix: Sign payload separately, attach signature
- [ ] Update `SignedPost` interface
- [ ] Update `createPost` function
- [ ] Add verification function

**Corrected Implementation**:
```typescript
interface SignedPost {
  type: 'post';
  postID: string;
  author: string;
  content: string;
  channelID: string;
  embedding: number[];
  timestamp: number;
  ttl: number;
  signature: Uint8Array;  // Signature of fields below
}

interface PostPayload {
  type: 'post';
  postID: string;
  author: string;
  content: string;
  channelID: string;
  embedding: number[];
  timestamp: number;
  ttl: number;
}

async function createPost(content: string, channelID: string): Promise<SignedPost> {
  const model = await loadEmbeddingModel();
  const embedding = await model.embed(content);

  const payload: PostPayload = {
    type: 'post',
    postID: generateUUID(),
    author: await getPeerID(),
    content,
    channelID,
    embedding,
    timestamp: Date.now(),
    ttl: 86400,
  };

  const signature = await sign(encode(payload), keypair.privateKey);

  return { ...payload, signature };
}
```

---

### 2.4 Profile Schema Ambiguity

**File**: `SOCIAL.md` (lines 185-213)

**Issue**: Is `bioEmbedding` stored or computed?

**Task**:
- [ ] Clarify: `bioEmbedding` is computed on-the-fly
- [ ] Add `computeBioEmbedding` function
- [ ] Update interface to mark as optional (computed field)

**Fix**:
```typescript
interface Profile {
  peerID: string;
  bio?: string;
  bioEmbedding?: number[];  // Computed: mean(channelEmbeddings)
  channels: ChannelSummary[];
  followerCount: number;
  followingCount: number;
  joinedAt: number;
}

async function computeBioEmbedding(profile: Profile): Promise<number[]> {
  if (profile.channels.length === 0) return [];
  const embeddings = profile.channels.map(c => c.latestEmbedding);
  return meanVector(embeddings);  // Element-wise mean
}
```

---

## Phase 3: Complete Missing Specifications (Week 3)

### 3.1 NAT Traversal Protocol

**File**: `PROTOCOL.md` (line 234)

**Issue**: `tryCircuitRelay` undefined.

**Task**:
- [ ] Add circuit relay protocol specification
- [ ] Specify relay peer discovery mechanism
- [ ] Add relay selection algorithm
- [ ] Specify relay authentication

**Specification** (to be added):
```markdown
### Circuit Relay Protocol

**Relay Discovery**:
1. Query DHT for peers advertising `/isc/relay/1.0` protocol
2. Maintain list of 10 known relays with latency measurements
3. Refresh list every 5 minutes

**Relay Selection**:
- Rank by: latency (60%), uptime (30%), geographic diversity (10%)
- Select top 3 relays for connection attempt

**Connection Flow**:
```
Peer A (behind NAT)          Relay              Peer B (behind NAT)
      │                        │                       │
      │─── Reserve Slot ──────▶│                       │
      │◀── Slot ID: abc ──────│                       │
      │                        │─── Reserve Slot ─────▶│
      │                        │◀── Slot ID: xyz ─────│
      │─── Connect abc ──────▶│                       │
      │                        │─── Connect xyz ─────▶│
      │◀═══════════════════════│══════════════════════▶│
      │                    Direct P2P Connection        │
```

**Fallback Chain**:
1. Direct WebRTC connection
2. Circuit relay via relay #1
3. Circuit relay via relay #2
4. Circuit relay via relay #3
5. Display error: "Cannot connect to peer (NAT traversal failed)"
```

---

### 3.2 Reputation Formula — MATHEMATICAL SPECIFICATION

**File**: `SECURITY.md` (line 94)

**Issue**: "30-day half-life" — no formula.

**Task**:
- [ ] Add exponential decay formula
- [ ] Specify base reputation value
- [ ] Specify interaction delta values
- [ ] Specify decay calculation interval

**Specification** (to be added):
```markdown
### Reputation Score Calculation

**Formula**:
```
R(t) = R₀ × e^(-λt) + Σ(interaction_delta × e^(-λ(t - t_interaction)))

Where:
- R(t) = reputation at time t
- R₀ = initial reputation (0.5 for new peers)
- λ = ln(2) / 30 days = 0.0231 per day (decay constant)
- t = time since last activity (days)
- interaction_delta = +0.1 for successful interaction, -0.2 for flagged interaction
```

**Implementation**:
```javascript
function calculateReputation(interactions: Interaction[], now: number): number {
  const lambda = Math.log(2) / 30;  // 30-day half-life
  const baseReputation = 0.5;
  
  let reputation = baseReputation;
  for (const interaction of interactions) {
    const ageDays = (now - interaction.timestamp) / (1000 * 60 * 60 * 24);
    const delta = interaction.successful ? 0.1 : -0.2;
    reputation += delta * Math.exp(-lambda * ageDays);
  }
  
  return Math.max(0, Math.min(1, reputation));  // Clamp to [0, 1]
}
```

**Decay Calculation**: Run every 24 hours for all active peers.
```

---

### 3.3 Conflict Resolution — CRDT SPECIFICATION

**Files**: All (missing)

**Task**:
- [ ] Add CRDT specification for concurrent channel edits
- [ ] Add CRDT for follow relationships
- [ ] Add CRDT for reputation scores (LWW-Register)
- [ ] Specify conflict detection mechanism

**Specification** (to be added to PROTOCOL.md):
```markdown
### Conflict-Free Replicated Data Types (CRDTs)

**Channel Edits (LWW-Map)**:
- Each field has (value, timestamp, peerID) tuple
- On conflict: highest timestamp wins; peerID as tiebreaker
- Merge: element-wise max of (timestamp, peerID)

**Follow Relationships (OR-Set)**:
- Add: `{follower, followee, timestamp}`
- Remove: `{follower, followee, timestamp, tombstone: true}`
- Merge: union of all adds and removes
- Query: exists if add present without matching remove

**Reputation Scores (LWW-Register)**:
- Single value with (value, timestamp) tuple
- On conflict: highest timestamp wins
- Updated only by mutual interaction confirmation

**Conflict Detection**:
- Vector clock per peer: `{peerID: sequenceNumber}`
- Include vector clock in all announcements
- Detect concurrent operations: neither clock dominates
```

---

### 3.4 Pagination Protocol

**Files**: All (missing)

**Task**:
- [ ] Add cursor-based pagination for DHT queries
- [ ] Specify page size limits by tier
- [ ] Add continuation token format
- [ ] Specify expiration for cursors

**Specification** (to be added to PROTOCOL.md):
```markdown
### Pagination Protocol

**Cursor Format**:
```typescript
interface QueryCursor {
  lastKey: string;      // Last DHT key returned
  lastValue: string;    // Last value hash (for consistency)
  timestamp: number;    // Cursor creation time
  signature: Uint8Array; // Signed by requesting peer
}
```

**Page Size Limits**:
| Tier | Max Page Size | Default |
|------|---------------|---------|
| High | 100 | 50 |
| Mid | 50 | 20 |
| Low | 20 | 10 |
| Minimal | 10 | 5 |

**Query Flow**:
1. Initial query: `getMany(key, { count: pageSize })`
2. If more results available, return cursor in response
3. Subsequent query: `getMany(key, { count: pageSize, cursor: <cursor> })`
4. Cursor expires after 5 minutes

**Response Format**:
```typescript
interface PaginatedResponse<T> {
  items: T[];
  cursor?: QueryCursor;  // Omitted if no more results
  hasMore: boolean;
}
```
```

---

### 3.5 Caching Strategy

**Files**: All (missing)

**Task**:
- [ ] Specify cache invalidation strategy
- [ ] Define TTL for cached results
- [ ] Specify memory budget per tier
- [ ] Add cache warming on reconnect

**Specification** (to be added to PROTOCOL.md):
```markdown
### Caching Strategy

**Cache Layers**:
1. **L1 (In-Memory)**: Hot data (active channels, recent matches)
2. **L2 (IndexedDB)**: Warm data (all channels, match history)
3. **L3 (DHT)**: Cold data (network-wide announcements)

**TTL by Data Type**:
| Data Type | L1 TTL | L2 TTL | Source of Truth |
|-----------|--------|--------|-----------------|
| Active channel distributions | 30s | 5min | Local compute |
| Match results | 60s | 10min | DHT query |
| Peer profiles | 5min | 1hr | DHT query |
| Mute lists | 1min | 10min | DHT query |
| Model registry | 1hr | 24hr | DHT query |

**Memory Budget**:
| Tier | L1 Budget | L2 Budget |
|------|-----------|-----------|
| High | 500MB | 2GB |
| Mid | 200MB | 1GB |
| Low | 100MB | 500MB |
| Minimal | 50MB | 200MB |

**Invalidation**:
- L1: Time-based expiry + LRU eviction
- L2: Time-based expiry + quota-based eviction
- DHT: TTL-based (enforced by DHT)

**Cache Warming**:
On reconnect after >5min offline:
1. Re-announce own channels
2. Re-query active channel matches
3. Refresh mute lists
4. Update model registry if >1hr old
```

---

### 3.6 Accessibility Specification

**Files**: All (one mention in SECURITY.md checklist)

**Task**:
- [ ] Add WCAG 2.1 AA compliance specification
- [ ] Specify screen reader compatibility
- [ ] Specify keyboard navigation
- [ ] Specify color contrast requirements
- [ ] Specify cognitive load considerations

**Specification** (new file `ACCESSIBILITY.md` or section in README.md):
```markdown
### Accessibility (WCAG 2.1 AA)

**Screen Reader Support**:
- All interactive elements have accessible names
- Live regions for dynamic content (match results, chat messages)
- Tested with: NVDA, VoiceOver, JAWS

**Keyboard Navigation**:
- Tab order follows visual layout
- All actions accessible via keyboard (no mouse-only interactions)
- Focus indicators visible (3px outline, 3:1 contrast)
- Skip links to main content

**Color Contrast**:
- Text: 4.5:1 minimum contrast ratio
- Large text (18px+): 3:1 minimum
- UI components: 3:1 minimum
- Tested with: axe, WAVE

**Cognitive Load**:
- Consistent navigation across all views
- Clear labels (no icons without text)
- Undo available for destructive actions
- Time limits adjustable (chat timeouts, match expiry)

**Testing**:
- Automated: axe-core in CI
- Manual: Screen reader testing quarterly
- User testing: Include users with disabilities in beta
```

---

## Phase 4: Fix Infeasibilities (Week 4)

### 4.1 Performance Claims — REVISE TO REALISTIC

**File**: `ROADMAP.md` (Success Criteria sections)

**Current (Infeasible)**:
- "Median time-to-first-match: <10s"
- "Connection failure rate: <5%"

**Task**:
- [ ] Revise time-to-first-match to 15-30s (model load + DHT + compute)
- [ ] Revise connection failure rate to 10-15% (realistic NAT traversal)
- [ ] Revise supernode latency target to <1000ms (embedding + network + crypto)
- [ ] Add device tier breakdown (High tier faster, Minimal tier slower)

**Revised Targets**:
```markdown
### Phase 1 Success Criteria (Revised)

| Metric | Original | Revised | Rationale |
|--------|----------|---------|-----------|
| Time-to-First-Match (High tier) | <10s | <15s | Model load (3-5s) + DHT (5-8s) + compute (1-2s) |
| Time-to-First-Match (Low tier) | <10s | <30s | Model load (10-15s) + DHT (10-15s) |
| Connection Failure Rate | <5% | <15% | Realistic NAT traversal without dedicated infra |
| Delegation Avg Latency | <500ms | <1000ms | Embedding (100-500ms) + network (50-200ms) + crypto (10-50ms) |
```

---

### 4.2 Scaling Claims — ADD CAVEATS

**File**: `README.md` (line 127)

**Current**: "Scales infinitely with users"

**Task**:
- [ ] Remove "infinitely" claim
- [ ] Add practical limits (DHT O(log n), browser memory, WebRTC connections)
- [ ] Specify expected scale per deployment mode

**Revised**:
```markdown
**Cost**: Zero server-side compute for core functionality.
**Scale**: DHT provides O(log n) lookup; practical limits:
- Browser memory: ~10k ANN index entries (HNSW)
- WebRTC connections: ~50 concurrent per browser
- Bootstrap bandwidth: Bottleneck at >100k concurrent users
**Mitigation**: Hierarchical DHT, community relays, federation (Phase 2+)
```

---

### 4.3 "Zero Server-Side Compute" — CORRECT CLAIM

**File**: `README.md` (multiple locations)

**Current**: "Zero server-side compute"

**Task**:
- [ ] Correct to "Minimal server-side compute"
- [ ] List required infrastructure (bootstrap peers, STUN/TURN, relays)
- [ ] Specify community-run infrastructure model

**Revised**:
```markdown
**Infrastructure**:
- Bootstrap peers: 5-10 community-run libp2p relays (required)
- STUN servers: Public (Google, Cloudflare) — free
- TURN servers: Community-run (optional, for hard NATs)
- Circuit relays: Community-run (optional, for fallback)

**Server-Side Compute**: Minimal — only relay traffic, no application logic.
**Cost**: ~$50-200/month for bootstrap peer bandwidth at 10k users.
```

---

### 4.4 "100% Unit Testable" — ADD CAVEAT

**File**: `CODE.md` (line 38)

**Current**: "100% unit testable in Node.js, browser, or any JS environment"

**Task**:
- [ ] Add caveat: requires mocking/polyfilling for Web Crypto API, IndexedDB
- [ ] Specify test environment setup
- [ ] List what's actually testable without mocks

**Revised**:
```markdown
**Testable**: Core logic (embedding math, LSH, matching, crypto abstraction) testable in Node.js.
**Requires Mocks**: Web Crypto API (use Node.js crypto module), IndexedDB (use LevelDB adapter).
**Test Coverage Goal**: 90% for @isc/core, 70% for @isc/protocol.
```

---

### 4.5 ZK Proofs — MOVE TO PHASE 4+

**File**: `ROADMAP.md` (Phase 3)

**Current**: "ZK proximity proofs — prove sim > threshold without revealing vector"

**Task**:
- [ ] Move to Phase 4+ (research phase)
- [ ] Add note: "Depends on ZK proof research maturation"
- [ ] Add alternative: "Optional vector reveal with user consent"

**Revised**:
```markdown
### Phase 4+ (Research)

| Feature | Status | Notes |
|---------|--------|-------|
| ZK proximity proofs | Research | Prove similarity > threshold without revealing vector. Depends on ZK proof research maturation (2028+). |
| Alternative: Optional vector reveal | Planned | User can consent to reveal vector to specific peers for enhanced matching. |
```

---

## Phase 5: Add Missing Features (Week 5)

### 5.1 Backup & Recovery Mechanism

**Files**: All (missing)

**Task**:
- [ ] Add social recovery specification (trusted friends hold key shards)
- [ ] Add encrypted cloud backup option
- [ ] Add hardware key support (YubiKey, etc.)
- [ ] Specify key rotation ceremony

**Specification** (to be added to SECURITY.md):
```markdown
### Key Backup & Recovery

**Social Recovery (Shamir's Secret Sharing)**:
1. Split private key into N shards (threshold K required)
2. Distribute shards to trusted peers
3. Recovery: Collect K shards, reconstruct key
4. Parameters: N=5, K=3 (any 3 of 5 friends can recover)

**Encrypted Cloud Backup**:
1. Encrypt private key with user passphrase (PBKDF2, 100k iterations)
2. Upload to user's cloud storage (iCloud, Google Drive, Dropbox)
3. Recovery: Download, decrypt with passphrase

**Hardware Key Support**:
1. Store private key on YubiKey / hardware wallet
2. Sign operations via hardware key API
3. Key never leaves hardware device

**Key Rotation**:
1. Generate new keypair
2. Re-encrypt all stored data with new key
3. Re-sign all announcements with new key
4. Announce key rotation to DHT (signed by old key)
5. Grace period: 30 days (both keys accepted)
```

---

### 5.2 Bootstrap Peer Operations

**File**: `PROTOCOL.md` (Bootstrap Peers section)

**Current**: Hardcoded peers with no operational spec.

**Task**:
- [ ] Add bootstrap peer selection criteria
- [ ] Add rotation mechanism
- [ ] Add health monitoring spec
- [ ] Add community-run bootstrap program

**Specification** (to be added):
```markdown
### Bootstrap Peer Operations

**Selection Criteria**:
- Uptime: >95% over 30 days
- Bandwidth: >100 Mbps up
- Geographic diversity: At least 3 continents
- Operator diversity: At least 3 independent operators

**Rotation**:
- Review bootstrap list quarterly
- Add 1 new peer, remove lowest-performing peer
- Announce changes 30 days in advance

**Health Monitoring**:
- Heartbeat: Every 60 seconds
- Metrics: Latency, packet loss, connection success rate
- Alert threshold: >5% failure rate over 1 hour

**Community Bootstrap Program**:
1. Apply: Submit peer info + metrics endpoint
2. Review: Community reviews application (GitHub PR)
3. Onboard: Added to bootstrap list for 30-day trial
4. Graduate: After 30 days with >95% uptime, permanent addition
```

---

### 5.3 TURN Server Infrastructure

**File**: `PROTOCOL.md` (Network Configuration section)

**Current**: "Public STUN/TURN servers" — no spec.

**Task**:
- [ ] Add TURN server deployment specification
- [ ] Add cost model
- [ ] Add community TURN program
- [ ] Add credential rotation spec

**Specification** (to be added):
```markdown
### TURN Server Infrastructure

**Deployment**:
- Software: coturn (open source)
- Ports: 3478 (STUN/TURN), 5349 (TLS), 49152-65535 (relay ports)
- Bandwidth: 1 Gbps up minimum
- Regions: US-East, EU-West, Asia-Pacific (3 regions minimum)

**Cost Model** (at 10k users, 10% require TURN):
- Bandwidth: 100 users × 1 Mbps × 24/7 = 100 Gbps-month ≈ $500/month
- Compute: 3 servers × $50/month = $150/month
- Total: ~$650/month

**Community TURN Program**:
1. Deploy: Operator deploys coturn with ISC config
2. Register: Submit endpoint + capacity
3. Verify: Community verifies uptime + latency
4. Compensate: Optional donations from tip jar

**Credential Rotation**:
- Short-term credentials (RFC 8656)
- Validity: 6 hours
- Generated by: Any bootstrap peer
- Verified by: TURN server via shared secret
```

---

### 5.4 Analytics & Monitoring

**Files**: All (missing)

**Task**:
- [ ] Add optional anonymous telemetry spec
- [ ] Add performance metrics dashboard
- [ ] Add error reporting mechanism
- [ ] Specify privacy-preserving analytics

**Specification** (to be added to SECURITY.md):
```markdown
### Analytics & Monitoring

**Optional Telemetry** (opt-in, privacy-preserving):
- Metrics: Time-to-first-match, connection success rate, delegation latency
- Aggregation: Daily aggregates, no individual tracking
- Anonymization: Differential privacy (ε=1.0)
- Export: Public dashboard (metrics.isc.network)

**Performance Metrics** (local, always-on):
- Model load time
- DHT query latency
- Match quality (similarity distribution)
- Available in DevTools Console

**Error Reporting** (opt-in):
- Stack traces (no PII)
- Context: Browser version, device tier, network state
- Aggregation: Frequency analysis, no individual tracking
- Export: GitHub Issues auto-filing for critical errors

**Privacy Guarantees**:
- No raw text or vectors transmitted
- No peer IDs in telemetry (hashed with daily salt)
- User can export/delete telemetry data
- User can disable telemetry at any time
```

---

## Phase 6: Economic Model (Week 6)

### 6.1 Sustainable Revenue Model

**File**: `SOCIAL.md` (line 243)

**Current**: "No ads: Monetization via crypto micropayments / Lightning Network tips (opt-in)"

**Task**:
- [ ] Add development cost projection
- [ ] Add tip volume projection (based on similar platforms)
- [ ] Add alternative revenue models
- [ ] Add grant/funding strategy

**Specification** (to be added to ROADMAP.md):
```markdown
### Economic Sustainability

**Development Costs** (annual):
- 3 developers × $100k = $300k
- Infrastructure (bootstrap, TURN) = $10k
- Legal, accounting, compliance = $40k
- Total: ~$350k/year

**Tip Volume Projection** (based on Nostr, Mastodon):
- Conversion rate: 2-5% of users tip
- Average tip: $5-10/month
- At 10k DAU: 200-500 tippers × $7.50 = $1.5k-3.75k/month
- Coverage: <5% of development costs

**Alternative Revenue**:
1. **Grants**: Protocol Labs, Ethereum Foundation, Mozilla ($50-200k/year)
2. **Enterprise Support**: Private deployments, SLA ($100-500k/year)
3. **Donations**: GitHub Sponsors, Open Collective ($10-50k/year)
4. **Supernode Hosting**: Managed supernode service ($5-20/month, 100 users = $6-24k/year)

**Phase 1-2 Strategy**: Grants + donations (community-funded)
**Phase 3+ Strategy**: Enterprise support + managed services (self-sustaining)
```

---

### 6.2 Supernode Economics

**File**: `ROADMAP.md` (Phase 3 Success Criteria)

**Current**: "Tips cover infrastructure for 20% of supernodes"

**Task**:
- [ ] Add pricing mechanism specification
- [ ] Add tip distribution algorithm
- [ ] Add projection of supernode costs
- [ ] Add sustainability analysis

**Specification** (to be added):
```markdown
### Supernode Economics

**Supernode Costs** (per month):
- Bandwidth: 50 Mbps up × 24/7 × $0.01/GB = ~$150/month
- Compute: 8-core server = ~$100/month
- Total: ~$250/month per supernode

**Tip Distribution**:
- 100% of tips go to supernode operators
- Platform takes 0% cut (Phase 1-3)
- Distribution: Proportional to uptime + requests served

**Sustainability Analysis**:
- At 10k DAU, 2% tip rate, $7.50 avg tip: $1.5k/month total
- Supports: $1.5k / $250 = 6 supernodes (3% of 200 supernodes)
- Gap: Tips cover 3%, not 20%

**Revised Target**:
- Phase 3: Tips cover 5% of supernode costs (realistic)
- Phase 4+: Enterprise support covers 50%+ of costs

**Alternative Incentives** (non-monetary):
- Reputation badges ("Trusted Supernode")
- Priority support (faster delegation responses)
- Governance rights (protocol upgrade voting)
```

---

## Phase 7: Final Verification (Week 6)

### 7.1 Cross-Document Consistency Check

**Task**:
- [ ] Verify all parameter values match across documents
- [ ] Verify all timelines match across documents
- [ ] Verify all interface definitions are single-source
- [ ] Verify all function signatures match

**Checklist**:
- [ ] Announcement TTL: Same value in README, PROTOCOL, SECURITY
- [ ] Similarity thresholds: Same ranges in README, SEMANTIC, PROTOCOL
- [ ] Rate limits: Same values in README, PROTOCOL, ROADMAP, test.md
- [ ] Model sizes: Same table in PROTOCOL, SEMANTIC
- [ ] Timeline: Same dates in ROADMAP, SOCIAL, README

---

### 7.2 Implementation Readiness Review

**Task**:
- [ ] All critical bugs fixed (LSH, spatiotemporal, crypto)
- [ ] All missing specs added (NAT, CRDT, pagination, caching)
- [ ] All infeasibilities corrected (performance, scaling, economics)
- [ ] All inconsistencies resolved (timelines, parameters)
- [ ] All edge cases covered (cold start, quota exceeded, NAT failure)

**Sign-off Required**:
- [ ] Technical lead: LSH implementation verified
- [ ] Security lead: Crypto implementation verified
- [ ] Protocol lead: NAT traversal verified
- [ ] Community lead: Accessibility spec reviewed

---

### 7.3 Documentation Quality Metrics

**Target Scores** (current average: 5.9/10):

| Document | Current | Target |
|----------|---------|--------|
| README.md | 6.7/10 | 8.5/10 |
| PROTOCOL.md | 5.7/10 | 9.0/10 |
| SEMANTIC.md | 5.0/10 | 9.0/10 |
| DELEGATION.md | 6.0/10 | 8.5/10 |
| SECURITY.md | 6.7/10 | 8.5/10 |
| SOCIAL.md | 5.0/10 | 8.0/10 |
| CODE.md | 6.0/10 | 8.5/10 |
| ROADMAP.md | 5.7/10 | 8.0/10 |

**Quality Criteria**:
- Completeness: All specs present, no gaps
- Consistency: No contradictions between documents
- Clarity: Unambiguous language, defined terms
- Feasibility: Realistic claims, validated assumptions

---

## Deliverables

### Week 1-2: Critical Fixes
- [ ] LSH implementation fixed and tested
- [ ] Spatiotemporal similarity complete
- [ ] Crypto implementation consistent
- [ ] Cold start mechanism specified
- [ ] Group chat thresholds defined
- [ ] Timeline contradictions resolved
- [ ] Parameter consistency table applied

### Week 3-4: Missing Specs
- [ ] NAT traversal protocol complete
- [ ] Reputation formula mathematical
- [ ] CRDT specification added
- [ ] Pagination protocol added
- [ ] Caching strategy added
- [ ] Accessibility specification added

### Week 5-6: Infeasibilities Fixed
- [ ] Performance claims realistic
- [ ] Scaling claims corrected
- [ ] "Zero server-side" claim corrected
- [ ] "100% testable" caveat added
- [ ] ZK proofs moved to Phase 4+
- [ ] Economic model added
- [ ] Supernode economics realistic

### Week 6: Verification
- [ ] Cross-document consistency verified
- [ ] Implementation readiness review complete
- [ ] Documentation quality targets met
- [ ] All original content recovered from archive

---

## Success Criteria

**Documentation Sprint Complete When**:

1. **No critical issues remain** (LSH, crypto, cold start, NAT, CRDT)
2. **All parameters consistent** across all documents
3. **All timelines realistic** and aligned
4. **All claims feasible** and validated
5. **Quality scores ≥ 8.0/10** average (from 5.9/10)
6. **Implementation team sign-off** (ready to code)

---

## Post-Sprint: Implementation Phase

**After documentation sprint completes**:

1. **Week 7-8**: Core extraction (@isc/core package)
2. **Week 9-12**: Adapter implementations (browser, Node.js)
3. **Week 13-16**: Protocol handlers (@isc/protocol)
4. **Week 17-20**: Browser app composition
5. **Week 21-24**: Testing, security audit, beta launch

**Total Timeline**: 6 weeks documentation + 18 weeks implementation = **24 weeks to Phase 1 beta**.
