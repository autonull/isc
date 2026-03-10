# ISC Getting Started Guide

> **Purpose**: Setup, usage, and local testing instructions.
>
> For architectural overview, see [README.md](README.md).

---

## Quick Start

> **No build step required for the MVP** — the app is a single HTML file importable via `<script type="module">`.

### 1. Clone the Repo

```bash
git clone https://github.com/yourname/isc.git
cd isc
```

### 2. Serve Locally

Any static server works:

```bash
# Using npx
npx serve .

# Or Python
python3 -m http.server 8080

# Or Node.js http-server
npx http-server -p 8080
```

### 3. Open in Browser

Open `http://localhost:8080` in two browser tabs (or two separate browsers).

### 4. Use ISC

In each tab:

1. **Create or pick a channel** — a default channel is created on first launch
2. **Type a description** of your current thoughts
3. **Optionally add relation tags** (location, time, mood…)
4. **Click Embed** — the model runs locally, producing your distribution
5. **Click Find Matches** — peers at nearby positions appear, ranked by similarity
6. **Click a match** to open a 1:1 chat, or let dense clusters auto-mesh into a group

> **First run**: The embedding model (~4–22 MB depending on tier) is downloaded and cached in IndexedDB. Subsequent loads skip the download entirely.

> **Testing locally**: Open two tabs, wait ~10 s for DHT bootstrap, then embed similar text in both. They should surface each other as a match.

---

## Device Tier Detection

ISC automatically detects your device capability at startup and selects an appropriate embedding model.

### Tier Override

Users can manually up/downgrade via Settings:

| Tier | Target | Model | Relations |
|------|--------|-------|-----------|
| **High** | Desktop/laptop | `all-MiniLM-L6-v2` (22 MB) | All (max 5) |
| **Mid** | Mid-range phone | `paraphrase-MiniLM-L3-v3` (8 MB) | Root + 2 |
| **Low** | Budget phone | `gte-tiny` (4 MB) | Root only |
| **Minimal** | Constrained | Word-hash fallback | Root only |

---

## Channels

### What is a Channel?

A **channel** is a named, editable presence context — your stated cluster of thoughts or interests at a given moment. You can maintain several channels simultaneously and switch between them.

```
┌─────────────────────────────────────────────────┐
│  Channels                          [+ New]       │
│ ─────────────────────────────────────────────── │
│  ● Work         "distributed systems, consensus" │
│  ○ Evening      "ambient music, slow fiction"    │
│  ○ Weekend      "ceramics, fermentation"         │
└─────────────────────────────────────────────────┘
```

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

**Rules**:
- Max 5 relations (UI-enforced)
- Tags must come from the [Relation Ontology](SEMANTIC.md#relation-ontology)
- Objects are free-form text or structured strings for spatiotemporal relations

### Channel Operations

| Operation | Description |
|---|---|
| **Create** | Click [+ New], enter name and description |
| **Switch** | Click channel name to activate |
| **Edit** | Click pencil icon to modify description/relations |
| **Duplicate** | Fork a channel to explore a related idea |
| **Archive** | Withdraw from DHT but keep locally |
| **Delete** | Permanently remove channel |

---

## Matching

### How Matching Works

ISC uses **approximate, ranked matching** — not discrete rooms:

1. **DHT Query**: Fetch candidates with matching LSH keys
2. **Local Refinement**: Rank by relational similarity score
3. **Display**: Show top-k matches above threshold

### Similarity Thresholds

| Range | Label | UI Treatment |
|---|---|---|
| 0.85+ | Very Close | Highlighted; auto-dial enabled |
| 0.70–0.85 | Nearby | Standard list entry |
| 0.55–0.70 | Orbiting | Dimmed; manual dial required |
| <0.55 | Distant | Filtered out by default |

### Match Actions

| Action | Description |
|---|---|
| **Dial** | Open 1:1 chat with match |
| **View Profile** | See match's public channels and posts |
| **Follow** | Subscribe to match's future posts |
| **Mute** | Block match from future results |

---

## Testing Supernode Delegation Locally

### 1. Start a Supernode

```bash
# In one terminal
npx serve . --port 8080

# Open http://localhost:8080?tier=high&supernode=true
# Enable "Supernode Mode" in Settings → Delegation
```

### 2. Start a Low-Tier Client

```bash
# In another terminal
npx serve . --port 8081

# Open http://localhost:8081?tier=low&delegate=true
# Enable "Use Delegation" in Settings → Delegation
```

### 3. Observe Delegation

- The low-tier client requests embedding assistance from the supernode
- Browser DevTools → Network tab shows encrypted delegation messages (protocol: `/isc/delegate/1.0`)
- Both peers should match semantically and form a chat within 30 seconds
- Verify in DevTools Console: `peer.delegation.stats` shows request/response counts

### 4. Test Fallback

- Close the supernode tab
- Low-tier client should gracefully degrade to local minimal model
- Match quality decreases but functionality remains

> **Note**: Local testing uses self-signed keys. Production delegation requires proper key management, reputation bootstrapping, and NAT traversal configuration.

---

## Troubleshooting

### No Matches Found

**Possible causes**:
- DHT bootstrap not complete (wait 10-30 seconds)
- Model mismatch (ensure both peers use same model tier)
- Similarity threshold too high (lower in Settings)
- No other peers online (open additional tabs)

**Solutions**:
1. Wait for DHT bootstrap (check status indicator)
2. Verify model tier matches across peers
3. Lower similarity threshold to 0.55
4. Open more tabs or invite other users

### Connection Failed

**Possible causes**:
- NAT/firewall blocking WebRTC
- Peer went offline
- Rate limit exceeded

**Solutions**:
1. Check browser console for WebRTC errors
2. Ensure STUN/TURN servers are accessible
3. Wait 60 seconds and retry
4. Try circuit relay option in Settings

### Model Download Stuck

**Possible causes**:
- Slow network connection
- Browser cache issues
- IndexedDB quota exceeded

**Solutions**:
1. Wait longer (model is 4-22 MB depending on tier)
2. Clear browser cache and reload
3. Check IndexedDB usage in DevTools
4. Try a lower tier model in Settings

### Delegation Not Working

**Possible causes**:
- No supernodes available
- Supernode rate limited
- Network partition

**Solutions**:
1. Check if any supernodes are advertised (DevTools Console)
2. Wait for supernode to become available
3. Fallback to local model should happen automatically
4. Report issue if persistent

---

## Keyboard Shortcuts

| Shortcut | Action |
|---|---|
| `Ctrl/Cmd + N` | New channel |
| `Ctrl/Cmd + E` | Embed current description |
| `Ctrl/Cmd + M` | Find matches |
| `Ctrl/Cmd + ,` | Open Settings |
| `Escape` | Close modal/dialog |
| `Ctrl/Cmd + K` | Command palette |

---

## Accessibility

ISC is designed for WCAG 2.1 AA compliance:

- **Screen reader tested**: NVDA, VoiceOver, JAWS
- **Keyboard navigation**: Full keyboard support
- **High contrast mode**: Available in Settings
- **Font size adjustment**: Available in Settings
- **Reduced motion**: Respects system preference

Report accessibility issues via GitHub Issues.

---

## Data Management

### Export Data

```
Settings → Privacy → Export Data
```

Downloads a JSON file containing:
- Keypair (encrypted if passphrase set)
- Channels
- Follows
- Mutes
- Chat history (if enabled)

### Delete Data

```
Settings → Privacy → Delete All Data
```

Permanently removes all local data:
- IndexedDB database
- localStorage entries
- Cached models

> **Warning**: This action cannot be undone. Export first if you want to preserve data.

---

## Network Configuration

### Bootstrap Peers

Default bootstrap peers (public libp2p relays):

```
/ip4/104.131.131.82/tcp/4001/p2p/QmaCpDMGvV2BGHeYERUEnRQAwe3N8SznbYGzPwpkqDrqEf
/ip4/104.131.131.82/tcp/4001/p2p/QmSoLju6m7xTh3DuokvT5887gRqQofnZ6Gqiq5KhCvv6ip
```

### Custom Bootstrap

```
Settings → Network → Custom Bootstrap Peers
```

Add your own bootstrap peers for private networks.

### STUN/TURN Servers

Default: Public STUN servers (Google, Cloudflare)

```
Settings → Network → Custom STUN/TURN
```

Add custom TURN servers for restrictive NATs.

---

## Browser Compatibility

| Browser | Version | Support |
|---|---|---|
| Chrome | 100+ | ✅ Full |
| Firefox | 100+ | ✅ Full |
| Safari | 15+ | ✅ Full |
| Edge | 100+ | ✅ Full |
| Opera | 90+ | ✅ Full |
| Brave | 1.40+ | ✅ Full |

**Required features**:
- Web Crypto API
- IndexedDB
- WebRTC
- Web Workers
- WASM

---

## Mobile Support

ISC works on mobile browsers with tier-appropriate models:

| Platform | Browser | Notes |
|---|---|---|
| iOS 15+ | Safari | Full support; model caching enabled |
| Android 10+ | Chrome | Full support; WebRTC optimized |
| Android 10+ | Firefox | Full support |

**PWA installation** (Phase 2):
- Add to home screen
- Offline-capable shell
- Push notifications (opt-in)

---

## Development Mode

### Local Development

```bash
# Install dependencies
npm install

# Start dev server
npm run dev

# Run tests
npm test

# Build for production
npm run build
```

### Debugging

Enable debug logging:

```
Settings → Developer → Enable Debug Logging
```

Console output includes:
- DHT announcements
- Match queries
- Delegation requests/responses
- WebRTC connection events

---

## Security Best Practices

1. **Enable passphrase encryption** for private keys (high-risk users)
2. **Backup your keypair** via Settings → Export
3. **Use ephemeral keys** for sensitive channels
4. **Review connected peers** periodically
5. **Report suspicious behavior** via the Report button

---

## Next Steps

- Read [README.md](README.md) for complete architectural specification
- Read [PROTOCOL.md](PROTOCOL.md) for P2P protocol details
- Read [SEMANTIC.md](SEMANTIC.md) for embedding and matching
- Read [SECURITY.md](SECURITY.md) for threat model and safety
- Read [SOCIAL.md](SOCIAL.md) for social network layer
- Read [ROADMAP.md](ROADMAP.md) for development timeline
