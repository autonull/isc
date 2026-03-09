# ISC Ultimate Semantic Model Extension Specification

## Vision and Overview
This specification reimagines the ISC (Internet Semantic Chat) semantic model to achieve **full relational semantic expressivity** within the project's constraints: browser-native execution, decentralized DHT-based discovery, and lightweight embeddings. The core limitation of prior designs—independent distributions composed post-hoc via logical operators—is transcended by embedding **relations intrinsically** into the model's geometry.

We shift from "filters on a root" to a **relational hypergraph** where channels are nodes, and optional **relations** (predefined tags like "in_location", "during_time", "causes_effect") bind concepts compositionally. This creates a meta-semantic space where "AI ethics in Neo-Tokyo during 2026" is not just filtered but **represented as a unified, discoverable embedding manifold**.

Key breakthroughs:
- **Predefined Relation Tags**: A fixed ontology of ~10 relations (expandable via community fork) ensures consistency without complexity explosion.
- **Compositional Embeddings**: Use prompt-engineered embeddings to fuse subject-relation-object triples into single vectors, enabling true binding (e.g., vector arithmetic approximates "X in Y" ≠ "Y in X").
- **Multi-Sample Discovery**: Break the "single hash" bottleneck by announcing multiple sampled points (one per active relation), with tiered fallbacks to single-root for low devices.
- **Expressivity Without Overhead**: Relations are optional; no relations → core ISC behavior. Discovery remains probabilistic via LSH, but relations boost recall for nuanced queries.

This design draws from emergent ideas in 2025-2026 AI (e.g., relational embeddings in Grok-3 variants, hypergraph LSH), but stays pure-JS implementable.

## Terminology
Minimal, unambiguous vocabulary—defined exhaustively:

- **Channel**: User-defined interest node with ID, name, and core text (description).
- **Distribution**: Gaussian in embedding space: mean vector (μ, 384D from all-MiniLM-L6-v2 or equivalent) + scalar spread (σ, 0.0–1.0 for fuzziness).
- **Root Distribution**: Primary semantic center from embedding the description + global σ.
- **Relation**: Optional binding between concepts, defined by:
  - **Tag**: Predefined string from ontology (e.g., "in_location", "during_time", "with_mood").
  - **Object**: Text or structured value (e.g., "Neo-Tokyo" or "lat:35.6895, long:139.6917").
  - **Fused Vector**: Composed μ from prompt: "subject [relation tag] object" (e.g., "AI ethics in_location Neo-Tokyo").
- **Ontology**: Fixed set of relation tags for interoperability:
  1. "in_location" (spatiotemporal place).
  2. "during_time" (temporal window).
  3. "with_mood" (emotional/tonal context).
  4. "under_domain" (categorical scope).
  5. "causes_effect" (causal link).
  6. "part_of" (compositional hierarchy).
  7. "similar_to" (analogical tie).
  8. "opposed_to" (contrastive relation).
  9. "requires" (dependency).
  10. "boosted_by" (amplification).
- **Hypergraph**: Conceptual model: Channel as node; relations as directed edges with fused vectors.
- **Announcement**: Multi-sampled points (one per fused vector) hashed via LSH for DHT put.
- **Matching**: Multi-vector similarity with relational alignment (e.g., graph edit distance approximation via vector dot products).

All terms are used precisely; no synonyms or expansions.

## Channel Schema
JSON object with optional `relations` array.

```json
{
  "id": "ch_ai_ethics_9b2f",
  "name": "AI Ethics",
  "description": "Ethical implications of machine learning and autonomy",  // Core text for root μ
  "spread": 0.15,  // Global σ
  "relations": [   // Optional; empty = root-only
    {
      "tag": "in_location",
      "object": "lat:35.6895, long:139.6917, radius:50km",  // Structured for accuracy
      "weight": 1.2  // Optional scalar emphasis (default 1.0)
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
- **Rules**: Max 5 relations (UI-enforced). Tags from ontology only. Objects: text for semantic, structured strings for spatiotemporal.

## Distribution Computation
Local browser process: Fuse relations into a set of distributions.

1. **Root**: Embed(description) → μ_root, σ_root = spread.
2. **Per-Relation Fused Distributions**:
   - For each relation:
     - Compose prompt: `${description} ${tag.replace('_', ' ')} ${object}` (e.g., "AI ethics in location lat:35.6895, long:139.6917, radius:50km").
     - Embed prompt → μ_fused.
     - For spatiotemporal tags: Pre-parse object (lat/long/radius or start/end) and append resolved string (e.g., "Neo-Tokyo area" via optional geocode).
     - σ_fused = spread * (1 / weight) for emphasis (higher weight → tighter).
3. **Hypergraph Representation**: Array of {root, fused1, fused2, ...} distributions.

Code sketch:
```javascript
async function computeRelationalDistributions(channel) {
  const extractor = await pipeline('feature-extraction', 'all-MiniLM-L6-v2');
  const rootEmb = await extractor(channel.description, { pooling: 'mean', normalize: true });
  const dists = [{ type: 'root', mu: Array.from(rootEmb.data), sigma: channel.spread }];

  if (channel.relations) {
    for (const rel of channel.relations) {
      let obj = rel.object;
      if (['in_location', 'during_time'].includes(rel.tag)) {
        const parsed = parseSpatiotemporal(rel.tag, obj);
        obj = `${obj} (${JSON.stringify(parsed)})`;  // Fuse structure into text
      }
      const prompt = `${channel.description} ${rel.tag.replace('_', ' ')} ${obj}`;
      const emb = await extractor(prompt, { pooling: 'mean', normalize: true });
      dists.push({
        type: 'fused',
        tag: rel.tag,
        mu: Array.from(emb.data),
        sigma: channel.spread / (rel.weight ?? 1.0),
        weight: rel.weight ?? 1.0
      });
    }
  }
  return dists;
}

// parseSpatiotemporal as in prior spec
```

This fuses relations semantically: The embedding model captures binding (e.g., "ethics in Neo-Tokyo" shifts μ differently than "Neo-Tokyo ethics").

## Announcement Process
Overcome single-hash limit with **multi-sample relational announcements**.

- **For each distribution** (root + fused): Sample point → LSH hash with prefix (e.g., "root:", "in_location:").
- **DHT Put**: Multiple entries per channel, but capped (e.g., max 5). Include metadata {peerID, channelID, relTag, sampleVec, ttl}.
- **Tier Fallback**: Low: Announce root only. Mid: Root + 2 key relations. High: All.
- **Optimization**: Use shared LSH params; compress vec via quantization if needed.

This enables relation-specific discovery: Query peers can LSH-search with prefixed keys for targeted recall.

## Matching Process
Relational alignment via multi-vector graph matching approximation.

1. **Candidate Collection**: LSH get with optional prefixes (e.g., fetch "in_location" samples first).
2. **Relational Score**:
   - Align graphs: For each my_dist (root/fused), find best-matching peer_dist via cosine(μ_my, μ_peer) * weight.
   - Aggregate: Weighted sum of alignments, normalized (soft graph isomorphism).
   - Spatiotemporal Boost: If tag is "in_location"/"during_time", add domain-specific similarity (Haversine/overlap) to the fused cosine.
   - Monte Carlo: Sample N from each dist, compute fraction of aligned samples.
3. **Threshold**: Match if aggregate > 0.75.

Code sketch:
```javascript
function relationalMatch(myDists, peerDists) {
  let score = 0;
  let totalWeight = 0;

  // Root alignment first
  const rootSim = cosineSimilarity(myDists[0].mu, peerDists[0].mu);
  score += rootSim;
  totalWeight += 1;

  // Fused alignments: Best-match bipartite
  for (let i = 1; i < myDists.length; i++) {
    let best = 0;
    for (let j = 1; j < peerDists.length; j++) {
      if (myDists[i].tag === peerDists[j].tag) {  // Prefer same-tag
        best = Math.max(best, cosineSimilarity(myDists[i].mu, peerDists[j].mu) * 1.2);
      } else {
        best = Math.max(best, cosineSimilarity(myDists[i].mu, peerDists[j].mu));
      }
    }
    if (['in_location', 'during_time'].includes(myDists[i].tag)) {
      const parsedMy = parseSpatiotemporal(myDists[i].tag, /* extract from prompt */);
      const parsedPeer = /* similar */;
      best += spatiotemporalSimilarity(myDists[i].tag, parsedMy, parsedPeer) * 0.5;  // Hybrid boost
    }
    score += best * myDists[i].weight;
    totalWeight += myDists[i].weight;
  }

  return (score / totalWeight) > 0.75;
}
```

## Implementation Notes
- **UI**: Relation builder with ontology dropdowns + object inputs (text or pickers).
- **Expressivity Examples**: "AI ethics causes_effect societal shifts" → fused vector captures causality. "Futurism opposed_to dystopia" → contrastive embedding.
- **Tiers & Limits**: Enforce max relations/samples to stay performant.
- **Privacy**: Relations ephemeral; multi-announce spreads load but reveals structure—mitigate with random tag salts if paranoid.

This ultimate design delivers true relational semantics: Bindings embedded compositionally, discoverable via multi-faceted LSH, matched as aligned hypergraphs. It transcends single-hash/filter limits, unlocking a semantically rich P2P network.
