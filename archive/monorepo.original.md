# ISC Monorepo Architecture

> **Purpose**: Define the modular architecture and package structure for ISC.

---

## Goal

Maximize code sharing across browser, Node.js server, CLI, and future form factors while maintaining clean separation of environment-specific concerns.

---

## Architecture Principles

1. **Core logic is environment-agnostic** — Embedding math, LSH, relational matching, and protocol logic have no browser or Node.js dependencies.
2. **Environment adapters are pluggable** — Storage, networking, and model loading are abstracted behind interfaces with multiple implementations.
3. **Dependencies are layered** — Core packages have zero browser-specific or Node.js-specific dependencies.
4. **Form factors are compositions** — Browser app, Node.js server, CLI, and future apps are thin compositions of shared packages.

---

## Package Structure

```
isc/
├── packages/
│   ├── core/                    # Environment-agnostic core logic
│   ├── adapters/                # Environment-specific implementations
│   ├── protocol/                # Libp2p protocol definitions
│   └── apps/                    # Form-factor compositions
│
├── docs/
│   ├── OVERVIEW.md
│   ├── PROTOCOL.md
│   ├── SEMANTIC.md
│   ├── DELEGATION.md
│   ├── SECURITY.md
│   ├── SOCIAL.md
│   ├── MONOREPO.md
│   ├── GETTING_STARTED.md
│   └── ROADMAP.md
│
├── package.json                 # Root workspace config
└── tsconfig.json                # Shared TypeScript config
```

---

## Package Details

### `@isc/core` — Environment-Agnostic Core

**Purpose**: Pure JavaScript/TypeScript logic with zero environment dependencies.

**Exports**:
```typescript
// Embedding & semantic matching
export function computeRelationalDistributions(channel: Channel): Distribution[];
export function relationalMatch(myDists: Distribution[], peerDists: Distribution[]): number;
export function cosineSimilarity(a: number[], b: number[]): number;

// LSH & DHT key generation
export function lshHash(vec: number[], channelId: string, seed: string): string[];

// Monte Carlo sampling
export function sampleFromDistribution(mu: number[], sigma: number, n: number): number[][];

// Cryptography (uses Web Crypto API abstraction)
export function generateKeypair(): Promise<Keypair>;
export function sign(payload: Uint8Array, keypair: Keypair): Promise<Signature>;
export function verify(payload: Uint8Array, signature: Signature, publicKey: PublicKey): Promise<boolean>;

// Types
export interface Channel { id: string; description: string; relations: Relation[]; spread: number; }
export interface Distribution { type: 'root' | 'fused'; tag?: string; mu: number[]; sigma: number; weight?: number; }
export interface Relation { tag: string; object: string; weight?: number; }
export interface Keypair { publicKey: PublicKey; privateKey: PrivateKey; }
```

**Dependencies**: None (pure JS/TS only)

**Testable**: 100% unit testable in Node.js, browser, or any JS environment.

---

### `@isc/adapters` — Environment-Specific Implementations

**Purpose**: Provide concrete implementations of abstract interfaces for different environments.

**Structure**:
```
adapters/
├── src/
│   ├── interfaces/              # Abstract interfaces
│   ├── browser/                 # Browser implementations
│   ├── node/                    # Node.js implementations
│   ├── cli/                     # CLI implementations
│   └── index.ts                 # Exports by environment
```

#### Interfaces (Abstract)

```typescript
// Storage interface
export interface StorageAdapter {
  get<T>(key: string): Promise<T | null>;
  set<T>(key: string, value: T): Promise<void>;
  delete(key: string): Promise<void>;
  keys(prefix?: string): AsyncIterable<string>;
}

// Model loading interface
export interface EmbeddingModelAdapter {
  load(modelId: string): Promise<void>;
  embed(text: string): Promise<number[]>;
  unload(): Promise<void>;
}

// Networking interface
export interface NetworkAdapter {
  announce(payload: SignedAnnouncement): Promise<void>;
  query(key: string): Promise<SignedAnnouncement[]>;
  dial(peerId: string, protocol: string): Promise<Stream>;
}

// Tier detection interface
export interface TierDetector {
  detect(): Promise<Tier>;
}
```

#### Browser Implementations

```typescript
// adapters/src/browser/index.ts
export const browserStorage: StorageAdapter = {
  get: async (key) => {
    const data = await indexedDB.get(key);
    return data ?? localStorage.getItem(key);
  },
  set: async (key, value) => { /* IndexedDB + localStorage fallback */ },
  // ...
};

export const browserModel: EmbeddingModelAdapter = {
  load: async (modelId) => {
    const { pipeline } = await import('@xenova/transformers.js');
    // Load model in Web Worker
  },
  embed: async (text) => { /* Web Worker inference */ },
  // ...
};

export const browserNetwork: NetworkAdapter = {
  announce: async (payload) => { /* libp2p browser */ },
  // ...
};

export const browserTierDetector: TierDetector = {
  detect: async () => {
    const cores = navigator.hardwareConcurrency ?? 2;
    const mem = navigator.deviceMemory ?? 1;
    // ...
  },
};
```

**Dependencies**: `@xenova/transformers.js`, `libp2p`, `indexeddb`, browser APIs

#### Node.js Implementations

```typescript
// adapters/src/node/index.ts
import { LevelDB } from 'leveldb';
import { createLibp2p } from 'libp2p';
import { tcp } from '@libp2p/tcp';
import { noise } from '@chainsafe/libp2p-noise';

export const nodeStorage: StorageAdapter = {
  get: async (key) => {
    return await db.get(key);
  },
  set: async (key, value) => { /* LevelDB/RocksDB */ },
  // ...
};

export const nodeModel: EmbeddingModelAdapter = {
  load: async (modelId) => {
    // Use @xenova/transformers.js in Node.js (ONNX Runtime)
  },
  embed: async (text) => { /* Native inference */ },
  // ...
};

export const nodeNetwork: NetworkAdapter = {
  announce: async (payload) => {
    const node = await createLibp2p({
      transports: [tcp()],
      connectionEncryption: [noise()],
      // ...
    });
  },
  // ...
};

export const nodeTierDetector: TierDetector = {
  detect: async () => {
    // Node.js: always 'high' tier (server has full resources)
    return 'high';
  },
};
```

**Dependencies**: `libp2p`, `@libp2p/tcp`, `leveldb`, `onnxruntime-node`

#### CLI Implementations

```typescript
// adapters/src/cli/index.ts
export const cliStorage: StorageAdapter = {
  get: async (key) => {
    const configPath = path.join(os.homedir(), '.isc', 'config.json');
    const config = JSON.parse(await fs.readFile(configPath, 'utf-8'));
    return config[key];
  },
  set: async (key, value) => { /* Write to ~/.isc/config.json */ },
  // ...
};

export const cliModel: EmbeddingModelAdapter = {
  // Same as Node.js but with progress bars
  load: async (modelId) => {
    const { createProgressBar } = await import('cli-progress');
    // Show progress bar during model download
  },
  // ...
};
```

**Dependencies**: `cli-progress`, `fs`, `os`, `path`

---

### `@isc/protocol` — Libp2p Protocol Definitions

**Purpose**: Define protocol handlers, message formats, and stream handlers.

**Exports**:
```typescript
// Protocol constants
export const PROTOCOL_CHAT = '/isc/chat/1.0';
export const PROTOCOL_DELEGATE = '/isc/delegate/1.0';
export const PROTOCOL_ANNOUNCE = '/isc/announce/1.0';

// Message types
export interface ChatMessage { channelID: string; msg: string; timestamp: number; }
export interface DelegateRequest { requestID: string; service: string; payload: Uint8Array; }
export interface DelegateResponse { requestID: string; result: Uint8Array; signature: Signature; }

// Protocol handlers
export function createChatHandler(stream: Stream): ChatProtocol;
export function createDelegateHandler(capabilities: Capabilities): DelegateProtocol;
export function createAnnounceHandler(dht: DHT): AnnounceProtocol;
```

**Dependencies**: `@isc/core`, `libp2p`, `it-pipe`

---

### `@isc/apps/*` — Form Factor Compositions

#### Browser App (`@isc/apps/browser`)

```typescript
// apps/browser/src/main.ts
import { computeRelationalDistributions, relationalMatch } from '@isc/core';
import { browserStorage, browserModel, browserNetwork, browserTierDetector } from '@isc/adapters/browser';
import { createChatHandler, createDelegateHandler } from '@isc/protocol';

async function main() {
  // Initialize with browser-specific adapters
  const app = new ISCApp({
    storage: browserStorage,
    model: browserModel,
    network: browserNetwork,
    tierDetector: browserTierDetector,
  });

  // Mount UI
  app.mount(document.getElementById('app'));
}
```

**Dependencies**: `@isc/core`, `@isc/adapters`, `@isc/protocol`, React/Vue (optional)

#### Node.js Server (`@isc/apps/node`)

```typescript
// apps/node/src/server.ts
import { computeRelationalDistributions, relationalMatch } from '@isc/core';
import { nodeStorage, nodeModel, nodeNetwork, nodeTierDetector } from '@isc/adapters/node';
import { createChatHandler, createDelegateHandler } from '@isc/protocol';
import express from 'express';

async function main() {
  // Initialize with Node.js-specific adapters
  const app = new ISCApp({
    storage: nodeStorage,
    model: nodeModel,
    network: nodeNetwork,
    tierDetector: nodeTierDetector,
  });

  // Enable supernode mode
  app.enableSupernode({
    maxConcurrentRequests: 10,
    rateLimitPerMinute: 30,
  });

  // Expose HTTP API for monitoring
  const http = express();
  http.get('/metrics', (req, res) => {
    res.json(app.getMetrics());
  });

  http.listen(3000);
}
```

**Dependencies**: `@isc/core`, `@isc/adapters`, `@isc/protocol`, `express`, `prom-client`

#### CLI App (`@isc/apps/cli`)

```typescript
// apps/cli/src/cli.ts
import { computeRelationalDistributions } from '@isc/core';
import { cliStorage, cliModel } from '@isc/adapters/cli';
import { Command } from 'commander';

const program = new Command();

program
  .command('embed <text>')
  .description('Embed text and print vector')
  .action(async (text) => {
    const model = cliModel;
    await model.load('Xenova/all-MiniLM-L6-v2');
    const embedding = await model.embed(text);
    console.log(JSON.stringify(embedding));
  });

program
  .command('match <text1> <text2>')
  .description('Compute similarity between two texts')
  .action(async (text1, text2) => {
    const model = cliModel;
    await model.load('Xenova/all-MiniLM-L6-v2');
    const [vec1, vec2] = await Promise.all([model.embed(text1), model.embed(text2)]);
    const sim = cosineSimilarity(vec1, vec2);
    console.log(`Similarity: ${sim.toFixed(4)}`);
  });

program.parse();
```

**Dependencies**: `@isc/core`, `@isc/adapters`, `commander`

#### Future: React Native App (`@isc/apps/react-native`)

```typescript
// apps/react-native/src/App.tsx
import { computeRelationalDistributions } from '@isc/core';
import { rnStorage, rnModel, rnNetwork, rnTierDetector } from '@isc/adapters/react-native';

function App() {
  const app = useMemo(() => new ISCApp({
    storage: rnStorage,     // AsyncStorage + SQLite
    model: rnModel,         // react-native-ml or TensorFlow Lite
    network: rnNetwork,     // react-native-libp2p
    tierDetector: rnTierDetector,
  }), []);

  return <ISCUI app={app} />;
}
```

**Dependencies**: `@isc/core`, `@isc/adapters`, `react-native`, `react-native-ml`

#### Future: Enterprise Server (`@isc/apps/enterprise`)

```typescript
// apps/enterprise/src/cluster.ts
import { nodeStorage, nodeModel, nodeNetwork } from '@isc/adapters/node';
import { createLibp2p } from 'libp2p';
import { redis } from 'ioredis';
import { postgres } from 'pg';

// Enterprise-specific features:
// - Redis-backed DHT for horizontal scaling
// - PostgreSQL for audit logging
// - Kubernetes service discovery
// - Prometheus metrics export
// - mTLS for inter-service auth

class EnterpriseISC {
  constructor(config: EnterpriseConfig) {
    this.storage = new RedisStorage(config.redis);
    this.audit = new PostgresAudit(config.postgres);
    this.network = new ClusteredLibp2p(config.kubernetes);
  }
}
```

**Dependencies**: `@isc/core`, `@isc/adapters`, `ioredis`, `pg`, `kubernetes-client`

---

## Dependency Graph

```
┌─────────────────────────────────────────────────────────────────┐
│                        Form Factor Apps                          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────────┐ │
│  │ Browser  │  │  Node.js │  │   CLI    │  │  React Native    │ │
│  │   App    │  │  Server  │  │   Tool   │  │      Mobile      │ │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────────┬─────────┘ │
│       │             │             │                  │           │
│       └─────────────┴─────────────┴──────────────────┘           │
│                              │                                    │
│                              ▼                                    │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                   @isc/protocol                              │ │
│  │            (Libp2p protocol handlers)                        │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                              │                                    │
│                              ▼                                    │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                  @isc/adapters                               │ │
│  │   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────────┐  │ │
│  │   │ Browser  │  │ Node.js  │  │   CLI    │  │    RN      │  │ │
│  │   └──────────┘  └──────────┘  └──────────┘  └────────────┘  │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                              │                                    │
│                              ▼                                    │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                    @isc/core                                 │ │
│  │         (Pure JS: embedding, LSH, matching, crypto)          │ │
│  └─────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

---

## Workspace Configuration

### Root `package.json`

```json
{
  "name": "isc-monorepo",
  "private": true,
  "workspaces": [
    "packages/core",
    "packages/adapters",
    "packages/protocol",
    "apps/*"
  ],
  "scripts": {
    "build": "turbo run build",
    "test": "turbo run test",
    "test:unit": "turbo run test --filter=@isc/core",
    "test:integration": "turbo run test --filter=@isc/apps/*",
    "lint": "turbo run lint",
    "dev:browser": "turbo run dev --filter=@isc/apps/browser",
    "dev:node": "turbo run dev --filter=@isc/apps/node",
    "dev:cli": "turbo run dev --filter=@isc/apps/cli"
  },
  "devDependencies": {
    "turbo": "^1.10",
    "typescript": "^5.0"
  }
}
```

### `packages/core/package.json`

```json
{
  "name": "@isc/core",
  "version": "0.1.0",
  "type": "module",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": "./dist/index.js",
    "./matching": "./dist/matching.js",
    "./crypto": "./dist/crypto.js"
  },
  "scripts": {
    "build": "tsc",
    "test": "vitest run"
  },
  "devDependencies": {
    "vitest": "^0.34",
    "typescript": "^5.0"
  }
}
```

### `packages/adapters/package.json`

```json
{
  "name": "@isc/adapters",
  "version": "0.1.0",
  "type": "module",
  "main": "./dist/index.js",
  "exports": {
    "./browser": "./dist/browser/index.js",
    "./node": "./dist/node/index.js",
    "./cli": "./dist/cli/index.js",
    "./interfaces": "./dist/interfaces/index.js"
  },
  "peerDependencies": {
    "@xenova/transformers.js": "^2.14",
    "libp2p": "^1.0"
  },
  "peerDependenciesMeta": {
    "@xenova/transformers.js": { "optional": true },
    "libp2p": { "optional": true }
  },
  "dependencies": {
    "@isc/core": "workspace:*"
  }
}
```

---

## Code Sharing Matrix

| Module | Browser | Node.js | CLI | React Native | Enterprise |
|--------|---------|---------|-----|--------------|------------|
| **@isc/core** | ✅ 100% | ✅ 100% | ✅ 100% | ✅ 100% | ✅ 100% |
| **@isc/protocol** | ✅ 100% | ✅ 100% | ⚠️ Partial | ✅ 100% | ✅ 100% |
| **Storage** | IndexedDB | LevelDB | JSON file | SQLite | Redis |
| **Model** | transformers.js | ONNX Runtime | ONNX Runtime | TFLite | ONNX Runtime |
| **Network** | libp2p (browser) | libp2p (TCP) | libp2p (TCP) | libp2p (RN) | libp2p (clustered) |
| **Tier detection** | Navigator API | Always 'high' | Always 'high' | React Native APIs | Always 'high' |

---

## Migration Strategy

### Phase 1: Extract Core

1. Move pure functions to `packages/core/`
2. Remove all browser/Node.js imports from core
3. Add comprehensive unit tests (run in Node.js CI)

### Phase 2: Create Adapters

1. Define interfaces in `adapters/src/interfaces/`
2. Move browser-specific code to `adapters/src/browser/`
3. Create Node.js implementations in `adapters/src/node/`
4. Update browser app to import from adapters

### Phase 3: Extract Protocol

1. Move libp2p protocol handlers to `packages/protocol/`
2. Define message types and handlers
3. Update both browser and Node.js apps to use protocol package

### Phase 4: Create Node.js Server

1. Compose `@isc/apps/node` from core + adapters + protocol
2. Add HTTP API for monitoring
3. Add Prometheus metrics export
4. Deploy as supernode reference implementation

### Phase 5: CLI Tool

1. Compose `@isc/apps/cli` for embedding/matching utilities
2. Add to npm for community use
3. Use in CI for ground-truth fixture generation

---

## Benefits

| Benefit | Description |
|---------|-------------|
| **Code reuse** | 80-90% of logic shared across all form factors |
| **Testability** | Core logic testable in Node.js without browser mocks |
| **Maintainability** | Bug fixes in core automatically benefit all apps |
| **Extensibility** | New form factors only need adapter implementations |
| **Performance** | Node.js server can use native ONNX Runtime (faster than WASM) |
| **Community** | CLI tool enables community experimentation without full app |
| **Enterprise** | Private instances can swap storage/network for enterprise-grade alternatives |

---

## Risks & Mitigations

| Risk | Mitigation |
|------|------------|
| **Package sprawl** | Enforce strict boundaries; core must have zero external deps |
| **Version drift** | Use workspace:* for internal deps; single version per release |
| **Adapter complexity** | Keep interfaces minimal; prefer composition over inheritance |
| **Build complexity** | Use Turborepo for caching and parallel builds |
| **Testing fragmentation** | Run core tests in CI for every PR; integration tests nightly |

---

## Example: Same Logic, Different Environments

### Browser

```typescript
import { computeRelationalDistributions } from '@isc/core';
import { browserModel } from '@isc/adapters/browser';

await browserModel.load('Xenova/all-MiniLM-L6-v2');
const dists = await computeRelationalDistributions(channel);
```

### Node.js Server

```typescript
import { computeRelationalDistributions } from '@isc/core';
import { nodeModel } from '@isc/adapters/node';

await nodeModel.load('Xenova/all-MiniLM-L6-v2');
const dists = await computeRelationalDistributions(channel);
```

### CLI

```typescript
import { computeRelationalDistributions } from '@isc/core';
import { cliModel } from '@isc/adapters/cli';

await cliModel.load('Xenova/all-MiniLM-L6-v2');
const dists = await computeRelationalDistributions(channel);
```

**Same core function, different model adapters, identical results.**

---

## Future Form Factors

| Form Factor | Adapter Needed | Effort |
|-------------|----------------|--------|
| **Desktop (Electron)** | Electron storage + model | Low (reuse Node.js) |
| **iOS/Android (Native)** | Swift/Kotlin core bindings | High (FFI layer) |
| **Edge (Cloudflare Workers)** | KV storage + edge model | Medium (WASM model) |
| **Browser Extension** | Extension storage + model | Low (reuse browser) |
| **Discord/Slack Bot** | Discord/Slack network adapter | Medium (protocol bridge) |
