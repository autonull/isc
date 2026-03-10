# ISC — UI Specification v2

> **Scope**: Complete use-case model and UI specification covering every interaction a user may perform across all phases of the ISC system. This document defines behavior at the product level; implementations should treat it as the single source of truth for user-facing decisions.

---

## Design Philosophy

ISC is built on a few principles that must permeate every screen and interaction:

| Principle | What it means for UI |
|---|---|
| **Thought-first, identity-second** | The primary unit is a *channel* (a thought context), not a user profile. Discovery starts from ideas, not people. |
| **Privacy by default** | Raw text is never sent unless the user explicitly posts. Vectors travel; text lives locally. |
| **Transparency without complexity** | Similarity scores, match reasons, and connection states are always visible, but never overwhelming. |
| **Progressive disclosure** | Minimal mode works out of the box; advanced controls (relations, chaos, delegation) reveal themselves contextually. |
| **Graceful degradation** | Every state — low-end device, poor network, no peers — has a natural, non-broken look. |
| **Serendipity is a feature** | The UI should feel exploratory. Encounters are discovered, not engineered. |

---

## Global State Model

Before enumerating screens, establish the states a peer can be in at any moment; every screen inherits from this:

| State | Description |
|---|---|
| **Bootstrapping** | Loading app shell, generating/restoring keypair, detecting device tier |
| **Model Loading** | Downloading/caching embedding model from CDN |
| **Connecting** | Joining DHT, reaching bootstrap peers |
| **Idle** | Connected; no active channels |
| **Announcing** | ≥1 channel active; embedding computed; DHT puts in flight |
| **Matching** | Query running; candidates ranked |
| **Chatting** | ≥1 open WebRTC stream |
| **Offline** | Network unavailable; queuing actions |
| **Degraded** | Minimal-tier or delegation-failed fallback mode active |

The persistent status bar at the bottom of every view shows the *least favorable* of these states.

---

## Screens & Views

```
App Shell
├── 1. Bootstrap / Onboarding
├── 2. Channel List (Home)
│   ├── 2a. Empty State
│   └── 2b. Active Channel List
├── 3. Channel Editor
│   ├── 3a. Create New Channel
│   └── 3b. Edit/Fork Channel
├── 4. Match Explorer
│   ├── 4a. Match List
│   ├── 4b. Peer Card (expanded)
│   └── 4c. Relation Map (2D vector view)
├── 5. Chat
│   ├── 5a. 1:1 Chat
│   ├── 5b. Group Chat
│   └── 5c. Direct Messages (DMs)
├── 6. Social Feed
│   ├── 6a. For You
│   ├── 6b. Following
│   └── 6c. Global Explore / Trending
├── 7. Post Composer
├── 8. Post Detail / Thread
├── 9. Profile View
│   ├── 9a. Own Profile
│   └── 9b. Peer Profile
├── 10. Communities
│   ├── 10a. Community List
│   ├── 10b. Community Channel View
│   └── 10c. Audio Space
├── 11. Notifications
├── 12. Settings
│   ├── 12a. Identity & Keys
│   ├── 12b. Privacy
│   ├── 12c. Matching
│   ├── 12d. Delegation
│   ├── 12e. Network
│   ├── 12f. Appearance & Accessibility
│   └── 12g. Developer / Debug
└── 13. CLI / Node.js Operator Dashboard (non-browser form factor)
```

---

## 1. Bootstrap / Onboarding

### 1.1 First Launch

**Entry condition**: No keypair in IndexedDB.

**Flow**:
1. App shell loads; shows ISC wordmark + tagline *"Meet your thought neighbors."*
2. **Capability probe** runs silently: `navigator.hardwareConcurrency`, `deviceMemory`, `connection.effectiveType`.
   - Tier determined; shown to user: *"Your device: High tier — full semantic matching enabled."*
   - Low/Minimal tier: *"Your device is limited. We'll use a helper node for best results."*
3. **Keypair generation**: `crypto.subtle.generateKey('Ed25519')` runs. A subtle spinner. On completion: *"Your identity is ready — no account needed."*
4. **Model download prompt** (if no cached model):
   - Shows: model name, size, estimated download time on current connection.
   - Options: **Download now** · **Download when on Wi-Fi** · **Skip (limited matching)**
   - On cellular with `saveData: true`: auto-defer with a banner.
5. **Invite / join prompt**:
   - If opened via invite link (`?peer=<peerID>&relay=<relayURL>`): auto-connects to the inviting peer; skips cold-start.
   - Otherwise: optional bootstrap peer list shown; user can proceed immediately.
6. **Optional passphrase** setup for key encryption.
   - Shown as a recommended step for high-risk users; can be skipped.
7. → Lands on **Channel List (Empty State)**.

### 1.2 Returning User

**Entry condition**: Keypair and channel data exist in IndexedDB.

**Flow**:
1. App shell loads; restores keypair.
2. Channels restored from localStorage.
3. **Cache warming** (if offline > 5 min):
   - Re-announce active channels.
   - Re-query match results.
   - Refresh mute lists.
   - Status bar shows: *"Syncing…"*
4. → Lands on **Channel List (Active State)**.

### 1.3 New Device / Import

**Entry condition**: User has existing keypair from another device.

**Actions available**:
- **Import via QR code**: camera scans a QR code encoding the encrypted keypair blob.
- **Import via file**: paste/upload an encrypted JSON export.
- **Social recovery**: enter peers' shard contributions (Shamir, N=5 K=3).
- After import: passphrase prompt (if key was passphrase-encrypted).

---

## 2. Channel List (Home)

The primary screen. A channel is the user's *current thought context*: a name, a description, optional relation tags, and a spread (fuzziness) value.

### 2.1 States

| State | What the user sees |
|---|---|
| **Empty** | Prompt: *"Describe what you're thinking about to find your thought neighbors."* + big **+ New Channel** button |
| **Active, no matches yet** | Channels listed; spinner next to each active channel; *"Searching…"* |
| **Active, matches found** | Match badge count next to channels; most-recent match timestamp |
| **Offline** | All badges grey; banner: *"You're offline. Matches paused."* |
| **Model loading** | Progress bar for model download in header |

### 2.2 Channel List Item

Each channel card shows:
- **Name** (large)
- **Description** (truncated to 2 lines)
- **Spread indicator**: a small fuzziness icon (σ = 0 → sharp, σ = 0.3 → blurred)
- **Relation tags**: up to 5 pill badges (e.g., `📍 Tokyo`, `🕓 2026`)
- **Match count**: *"3 nearby · 5 orbiting"*
- **Active/paused toggle**: tap to pause announcing (dims the card)
- **Last match**: relative timestamp

### 2.3 Actions from Channel List

| Action | Trigger | Behavior |
|---|---|---|
| **Open Match Explorer** | Tap channel card | Goes to Match Explorer for that channel |
| **Create channel** | `+` FAB or header button | Opens Channel Editor (create mode) |
| **Edit channel** | Long-press or swipe → Edit | Opens Channel Editor (edit mode) |
| **Fork channel** | Long-press or swipe → Fork | Duplicate channel; new ID; independent announce |
| **Archive channel** | Long-press or swipe → Archive | DHT TTL expires naturally; channel hidden from list |
| **Delete channel** | Long-press or swipe → Delete | Confirmation dialog; removes from localStorage |
| **Pause/resume announcing** | Toggle on card | Stops/starts DHT announcements; preserves embed |
| **Reorder channels** | Drag-and-drop | Persisted sort order in localStorage |
| **Switch active channel** | Tap channel (if multi-channel quick-switch) | Sets query context for Match Explorer |

### 2.4 Channel Count Limit

- Maximum **5 channels** active simultaneously (UI-enforced).
- Attempting to create a 6th active channel shows: *"Deactivate one channel to activate a new one."*
- Archived channels don't count toward the limit.

---

## 3. Channel Editor

### 3.1 Create New Channel

**Fields**:

| Field | Type | Notes |
|---|---|---|
| **Name** | Short text (≤ 50 chars) | Required. Shown in list; not embedded. |
| **Description** | Long text | Required. This is what gets embedded. Placeholder: *"Describe what you're thinking about…"* |
| **Spread (σ)** | Slider, 0.0–0.3 | Default 0.1. Tooltip: *"Higher spread = fuzzier matching; higher serendipity."* |
| **Relations** | Expandable section | 0–5 relation tags |

**Relation Tag Builder** (for each relation):
- **Tag picker**: dropdown of 10 canonical tags with plain-English labels and icons.
  - `in_location` → 📍 *Place*
  - `during_time` → 🕓 *Time*
  - `with_mood` → 🎭 *Mood*
  - `under_domain` → 🏷 *Domain*
  - `causes_effect` → ⚡ *Causes*
  - `part_of` → 🧩 *Part of*
  - `similar_to` → 🔗 *Similar to*
  - `opposed_to` → ↔ *Opposed to*
  - `requires` → 🔑 *Requires*
  - `boosted_by` → 🚀 *Boosted by*
- **Object field**: free text for most tags; structured picker for `in_location` (map pin or lat/long) and `during_time` (date-range picker).
- **Weight**: optional numeric input (default 1.0); shown only on expand (*"Advanced"*).

**Live preview**:
- As user types, show: *"Embedding will capture: [description] [tag] [object]"* for each relation.
- Similarity preview: if user has any other channels, show estimated semantic distance to them.

**Actions**:
- **Save & Activate**: embeds, announces to DHT, navigates to Match Explorer.
- **Save Draft**: saves without activating.
- **Cancel**: discard changes, confirm if unsaved.

### 3.2 Edit Channel

Identical to Create, but pre-populated. On save:
- Re-computes embedding.
- Issues new DHT put within 5 seconds.
- Old vectors expire via TTL naturally.

### 3.3 Fork Channel

Creates a new channel with a copied description and relations, unique ID. Both can diverge independently.

---

## 4. Match Explorer

The discovery view for a single channel. Shows who is semantically nearby.

### 4.1 Match List (Default)

**Header**: Channel name + spread indicator. Active indicator (pulsing dot = announcing).

**Controls row**:
- **Threshold slider**: sets minimum similarity shown (default 0.55; range 0.30–0.95).
- **Sort**: Similarity (default) · Freshness · Mutual relations count.
- **Filter by relation tag**: show only peers sharing a specific tag.
- **Chaos mode toggle**: perturbation dial (0.0–0.3) for serendipity.
- **Refresh**: manual re-query (auto-refreshes on tier schedule).

**Match List**:

Each match entry shows:
- **Similarity score**: large numeric badge with color gradient (0.85+ = green, 0.70–0.85 = teal, 0.55–0.70 = grey).
- **Proximity label**: "Very Close" / "Nearby" / "Orbiting".
- **Peer ID**: abbreviated (first + last 4 chars); no name unless they've posted.
- **Shared relation tags**: small badges showing which tags overlap.
- **Connection state**: online indicator; last seen.
- **Supernode badge**: if the peer is a trusted supernode.
- **Actions**: **Dial** button (prominent at 0.85+; subdued at 0.55–0.70) · **View Profile** · **Mute**.

**Thresholds and auto-dial**:
- **0.85+**: "Very Close" — **Dial** CTA is bright; auto-dial prompt offered ("Auto-connect now?").
- **0.70–0.85**: "Nearby" — Standard **Dial** button.
- **0.55–0.70**: "Orbiting" — Dimmed; tap to expand and manually dial.
- **< 0.55**: Filtered out by default; toggle to reveal.

**Empty state per tier**:
- *"No peers in your neighborhood yet. Stay here and let your thoughts propagate."*
- If Minimal tier and no delegation: *"Using word-matching mode — connect to a Wi-Fi network to upgrade."*

### 4.2 Peer Card (Expanded)

Tapping a match expands to a detailed card:
- **Full similarity breakdown**: root score + each fused relation score labeled.
- **Matched relations**: side-by-side comparison of your relations vs. theirs.
- **Explainability statement**: *"You both expressed interest in AI ethics during 2026 in Tokyo."*
- **Thought Bridge** (Phase 3+): AI-suggested conversation opener bridging the two embeddings geometrically.
- **Actions**: **Dial Chat** · **View Profile** · **Follow** · **Mute** · **Report**.

### 4.3 Relation Map (2D Semantic View)

A 2D projection (PCA or UMAP-lite) of the peer embeddings relative to the user's channel:
- Users own channel plotted at center.
- Each peer dot, colored by proximity tier.
- Tap a dot → opens Peer Card.
- Pinch-to-zoom; drag to pan.
- Chaos mode: shows perturbed position alongside original (ghost dot).

**Toggle**: List ↔ Map toggle in header.

---

## 5. Chat

### 5.1 1:1 Chat

**Entry**: Via **Dial** from Match Explorer.

**Header**: Similarity score badge · Peer ID · Connection latency indicator · Mute · Close.

**Message area**:
- Messages appear as bubbles (own right, peer left).
- Each message shows: timestamp · delivery indicator (sent / delivered / read if future).
- Signatures verified silently; tampered messages show a red ⚠️ badge.

**Input area**:
- Text input with **Send** button.
- Emoji picker.
- **Thought Bridge suggestion** (if enabled in settings): AI-generated opener shown as a ghost prompt; tap to adopt.
- **File attachment** (Phase 3): media via IPFS.

**Connection state banners**:
- *"Connecting via relay…"* (circuit relay fallback).
- *"Connection lost — retrying…"* (reconnect wait).
- *"Peer drifted — similarity now 0.38"* (thought drift warning; soft exit prompt).

**Actions from header**:
- **View peer's profile**.
- **Invite to group** (if 3+ mutually proximal peers are online).
- **Mute** (one-click; propagates signed mute event to DHT).
- **Report** peer.
- **Close / hang up** (ends WebRTC stream; does not mute).

**Automatic exit prompt**: when peer's similarity drops below 0.55: *"Your thoughts are drifting apart. End this chat?"* — with Dismiss / End Chat options.

### 5.2 Group Chat

**Entry**: Auto-formed when 3+ peers share pairwise similarity ≥ 0.85; or manually invited.

**Header**: Shows group member count + room ID (hash of centroid) + member list icon.

**Member panel**: Collapsible panel showing each member's similarity to the group centroid; leaves a dot when they drift below threshold.

**Invitation received**: In-app toast: *"Peer QmX… invited you to a group — similarity 0.91. [Join] [Ignore]"*

**Latecomer join**: User can join via centroid key (shown in group header or share link).

**Group exit**: Graceful exit prompt when drifting below 0.55. Exit is announced and logged locally.

**Maximum size**: 8 peers (UI shows *"Full"* when limit reached; observer mode in Phase 3).

### 5.3 Direct Messages (DMs)

**Entry**: Via Peer Profile → **Message** or from a post's action menu.

**Header**: Peer ID + E2E encrypted badge.

**Composition**: Identical to 1:1 chat. Content is encrypted via libsodium sealed box before leaving the browser.

**Group DMs**: Up to the WebRTC mesh limit. Created from a peer group or community member list.

**DM notifications**: Badge on the Notifications icon; no preview text shown in system notifications (privacy).

---

## 6. Social Feed (Phase 3+)

Three tab views accessible from the bottom bar.

### 6.1 For You Feed

Posts ranked by semantic proximity to active channels.

**Each post card shows**:
- **Author**: abbreviated peer ID (name if they've published a bio).
- **Content**: full text (≤ 280 chars) or truncated long-form with *"Read more"*.
- **Similarity badge**: *"0.82 similar to your AI Ethics channel"* — always present.
- **Matched channel**: which of the user's channels triggered the match.
- **Timestamp** + relative time.
- **Engagement counts**: ❤️ likes · 🔁 reposts · 💬 replies.
- **Actions**: Like · Repost · Reply · Quote · Share · Report · Mute Author.

**Feed controls**:
- **Semantic threshold slider**: minimum similarity to appear in feed (default 0.6).
- **Diversity toggle**: enforce a minimum serendipity fraction (enables Chaos mode on queries).
- **Refresh**: manual; auto-refresh every 5 minutes.

**Empty state**: *"No nearby posts yet. Create a post or expand your threshold to discover more."*

### 6.2 Following Feed

Chronological posts from explicitly followed peers.

**Controls**: Sort by time (default) or by similarity. Filter by followed peer.

**Empty state**: *"Follow peers from the Match Explorer to see their posts here."*

### 6.3 Global Explore / Trending

High-engagement post clusters; updated hourly via DHT trending key.

**Sections**:
- **Trending Now**: top 20 posts by weighted engagement (replies > reposts > likes) in the last hour.
- **Semantic Clusters**: named topic clouds based on vector density in embedding space.
- **Rising**: posts gaining engagement rapidly in the last 15 minutes.

**Filters**: time window (1h / 6h / 24h); model shard (only shows posts in the user's model space).

---

## 7. Post Composer

**Entry**: FAB from Feed views or via share from chat.

**Fields**:
| Field | Notes |
|---|---|
| **Content** | Multiline textarea. ≤ 280 chars for short post; unlimited for long-form (long-form collapses in feed). Character count displayed. |
| **Channel** | Which of your channels to embed under. Drives DHT routing. |
| **Attach media** (Phase 3) | Upload via IPFS. Image / video / audio. |
| **Long-form mode** | Toggle to remove character limit. Shows estimated read time. |

**Semantic preview**:
- As user types, shows real-time embedding similarity to their selected channel.
- Warning if *"Post is off-topic from your channel (similarity 0.30)"*.

**Quote mode**: If quoting a post, original is shown inline. Fused embedding is computed from original + commentary.

**Drafts**: Autosave to IndexedDB every 10 seconds. Accessible from Composer header → *"Drafts"*.

**Actions**:
- **Post**: signs, announces to DHT.
- **Schedule** (Phase 3): queue for later with a datetime picker.
- **Discard**: confirmation if content is non-empty.

---

## 8. Post Detail / Thread

**Entry**: Tapping a post card.

**Structure**:
- **Original post** (full text).
- **Similarity explanation**: which channel matched and score.
- **Embedded vector indicator**: small cluster map showing post position relative to your channel (optional; toggle in settings).
- **Replies**: threaded; each reply shows its own similarity to the original post.
- **Engagement summary**: Like counts, repost count, quote count.
- **Quote tweets**: listed below replies.

**Actions on original post and replies**: Like · Reply · Repost · Quote · Share link · Report · Mute Author.

**Share link**: `https://isc.network/post/<postID>` — resolves via DHT on load.

---

## 9. Profile View

### 9.1 Own Profile

**Access**: Bottom bar → Profile icon or Settings → Profile.

**Content**:
- **Peer ID** (full + abbreviated + copy button).
- **Bio**: optional free-text bio. Embedded as mean of channel embeddings if not set explicitly.
- **Active channels**: list with spread, relation tags, match count.
- **Post history**: posts sorted by timestamp; count shown.
- **Follower count / Following count**.
- **Reputation score** (Phase 2+): shown as a meter; tooltip explains the decay formula.
- **Supernode badge**: if actively serving as a supernode.

**Actions**:
- **Edit bio**.
- **Export data** → JSON export of keypair, channels, history, follows (see Data Sovereignty).
- **Import data**.
- **Key management** → opens 12a. Identity & Keys settings.
- **Share profile**: generates invite link or QR code embedding peer ID + relay.

### 9.2 Peer Profile

**Content**:
- **Peer ID**.
- **Bio** (if published).
- **Channels** (if publicly announced): listed with names and descriptions.
- **Mutual channels**: channels where both users are active.
- **Similarity score**: to each of your active channels.
- **Follower count / Following count** (approximate, from DHT).
- **Reputation score** (Phase 2+).
- **Supernode status**.

**Actions**:
- **Follow / Unfollow**.
- **Chat / DM**.
- **Mute**: signed mute event; propagated to DHT.
- **Report**: reason picker + description field.
- **Block** (stronger than mute; local only, no DHT event).

---

## 10. Communities (Phase 3+)

### 10.1 Community List

Communities are **shared channels**: groups of peers co-owning a channel's mean/spread.

**Each community card shows**:
- **Name** + **description**.
- **Member count** (approx from DHT).
- **Similarity** to the user's active channels.
- **Your role**: Member / Co-editor / Viewer.

**Actions**: Join · Create community.

**Join flow**: Tapping a community shows a preview (description, spread, semantic cluster map). **Join** subscribes via libp2p pubsub; user's announcements are fused into community's mean vector.

**Create flow**: Enter name + description. Optionally invite co-editors (by peer ID or share link). Community channel is announced to DHT.

### 10.2 Community Channel View

Single community context:
- **Posts** feed filtered to community semantic space.
- **Members** tab: list of members with similarity scores + roles.
- **Edit** tab (co-editors only): adjust description, spread, add/remove relations.
- **Invite**: share link or QR code.
- **Leave**: removes from local follow list; no notification to community (pseudonymous membership).

**Semantic moderation**: Off-topic posts (similarity < 0.4 to community channel) are naturally deprioritized. Co-editors can report posts via signed `CommunityReport`.

### 10.3 Audio Space

**Entry**: Community Channel → **Start Audio Space** (or Join if one is active).

**UI**:
- Grid of participant avatars (peer ID abbreviations).
- Speaking indicator (waveform animation on active speaker's card).
- Mute self button (large, bottom center).
- Leave button.
- Listener count badge.
- Chat sidebar (text messages alongside audio).

**Behavior**: WebRTC mesh audio; up to 10 participants (UI-enforced). Beyond 10: observer mode (listen only, no audio send).

**Entry for listeners**: If an Audio Space is active in the community, a banner appears at the top of the Community Channel View: *"Live: 6 speaking. [Join]"*.

---

## 11. Notifications

**Types of notifications**:

| Event | Message | Channel |
|---|---|---|
| New match found | *"New match in [Channel]: 0.88 similarity"* | In-app badge |
| Group chat invite | *"QmX… invited you to a group chat"* | In-app toast |
| 1:1 chat dial | *"QmX… is calling (0.91 similarity)"* | In-app toast + sound |
| DM received | *"New message"* (no content preview) | In-app badge |
| Post reply | *"Someone replied to your post"* | In-app badge |
| Like / repost | *"Your post was liked / reposted"* | In-app badge (batched) |
| Follow | *"A peer followed you"* | In-app badge |
| Peer drifted | *"Chat partner's similarity dropped to 0.38"* | In-chat banner |
| Model migration | *"A new embedding model is available — update for best results"* | Persistent banner |
| Supernode offer | *"You're eligible to run a supernode"* | One-time nudge |
| Key backup reminder | *"Back up your identity key"* | Periodic reminder |
| Rate limit hit | *"Too many announces — slowing down"* | Status bar flash |
| DHT quota exceeded | *"Network quota reached — matching slowed"* | Status bar flash |

**Notification center**: Tapping the notification icon opens a full list sorted by time. Each item is dismissable individually or all-at-once.

**Quiet hours**: Time-based muting in Settings → Notifications.

---

## 12. Settings

### 12a. Identity & Keys

| Setting | Type | Description |
|---|---|---|
| **My Peer ID** | Display + Copy | Full peer ID |
| **Public key** | Display + Copy | Base58 encoded |
| **Passphrase encryption** | Toggle + Set passphrase | Encrypts IndexedDB keypair at rest |
| **Change passphrase** | Action | PBKDF2 re-derive + re-encrypt |
| **Key backup → Export** | Action | Download encrypted JSON file |
| **Key backup → QR code** | Action | Show QR of encrypted keypair |
| **Key backup → Social recovery** | Action | Split into 5 shards; share 3 with trusted peers |
| **Key recovery → Import** | Action | File / QR / Shard assembly |
| **Key rotation** | Action | Generate new keypair; re-sign channels; 30-day grace |
| **Ephemeral keys** | Toggle per channel | New throwaway keypair per channel session |
| **Hardware key (YubiKey)** | Connect | Delegate signing to hardware key (Phase 4) |

### 12b. Privacy

| Setting | Type | Description |
|---|---|---|
| **Announcement mode** | Picker: Normal / Ephemeral / Silent | Ephemeral: rotate peer ID each session; Silent: receive-only |
| **Channel spread default** | Slider (0.0–0.3) | Sets default σ for new channels |
| **IP obfuscation** | Toggle | Route via circuit relays; hides direct IP |
| **Tor / I2P routing** | Toggle (if plugin installed) | Routes all libp2p traffic via Tor or I2P |
| **Opt-in telemetry** | Toggle | Aggregated, diff-privacy metrics; default off |
| **Error reporting** | Toggle | Stack traces (no PII); default off |
| **Export my data** | Action | Full JSON export of all local data |
| **Delete all data** | Action | Wipes IndexedDB + localStorage; shows consequences |

### 12c. Matching

| Setting | Type | Description |
|---|---|---|
| **Default similarity threshold** | Slider (0.30–0.95) | Minimum to appear in match list; default 0.55 |
| **Auto-dial threshold** | Slider | Similarity above which auto-dial prompt appears; default 0.85 |
| **Max matches shown** | Number | Top-k limit; default 20; max 100 (High tier) |
| **Chaos level** | Slider (0.0–0.3) | Default perturbation for serendipity; 0 = off |
| **Relation weight: spatiotemporal boost** | Slider | Multiplier for `in_location` / `during_time` bonus |
| **Model tier override** | Picker | Manual override of auto-detected tier |
| **Show similarity scores** | Toggle | Hide/show numeric scores (for those who find them distracting) |
| **Show orbit zone** | Toggle | Show "Orbiting" (0.55–0.70) peers; default on |
| **Announce on cellular** | Toggle | Default off; shows size warning |

### 12d. Delegation

| Setting | Type | Description |
|---|---|---|
| **Use delegation** | Toggle | Allow supernode assistance; default on for Low/Minimal |
| **Prefer local compute** | Toggle | Try local first; delegate only on failure; default on |
| **Trusted supernodes** | List + Add | Manually pin specific peer IDs as preferred supernodes |
| **Max delegation latency** | Slider (500–10000 ms) | Reject responses slower than this threshold |
| **Max delegations per minute** | Number | Client-side rate cap; default 3 |
| **Enable supernode mode** | Toggle | Serve other peers; only offered on High-tier devices |
| **Supernode max concurrent** | Number (if supernode enabled) | Max parallel requests serviced |
| **Advertise uptime** | Toggle (if supernode enabled) | Include uptime in capability announcement |
| **Accept tips** | Toggle (if supernode enabled) | Lightning Network tip address |
| **Lightning address** | Text (if tips enabled) | E.g., `user@getalby.com` |
| **Per-channel delegation control** | Per-channel toggles | Disable delegation for sensitive channels |

### 12e. Network

| Setting | Type | Description |
|---|---|---|
| **Bootstrap peers** | List + Add/Remove | Custom bootstrap peer multiaddresses |
| **TURN servers** | List + Add/Remove | Custom TURN server URIs + credentials |
| **Max concurrent connections** | Number | WebRTC connection cap; default 50 |
| **Heartbeat interval** | Slider (10–120 s) | Default 30 s |
| **Reconnect delay** | Slider (1–60 s) | Default 5 s |
| **Announce interval override** | Picker (off / per-tier) | Manual override of auto refresh interval |
| **Debug logging** | Toggle | Print DHT, match, delegation, WebRTC events to console |
| **Network status** | Display | Connected peers count, DHT key count, relay status |
| **Seed tab** | Action | Open a dedicated tab that stays connected as a local bootstrap |
| **Direct connect** | Action | Enter a peer ID + relay URL to dial manually |
| **QR connect** | Action | Scan a peer's QR code to dial directly |

### 12f. Appearance & Accessibility

| Setting | Type | Description |
|---|---|---|
| **Theme** | Picker: System / Light / Dark / High Contrast | |
| **Font size** | Slider | Relative font scaling |
| **Reduce motion** | Toggle | Disables micro-animations; respects `prefers-reduced-motion` |
| **Screen reader mode** | Toggle | Enhances ARIA live regions verbosity |
| **Focus outline** | Toggle | Always-visible 3px focus ring (default on) |
| **Language / locale** | Picker | UI language; affects date/number formatting |
| **RTL layout** | Auto | Activated by Arabic, Hebrew, Persian locale selection |
| **Compact mode** | Toggle | Denser channel list and match list |
| **Show vector map** | Toggle | 2D semantic scatter plot in Match Explorer; can hide for performance |
| **Match card density** | Picker: Comfortable / Compact | |

### 12g. Developer / Debug

| Setting | Type | Description |
|---|---|---|
| **Device tier override** | Picker | Force a specific tier for testing |
| **Debug logging** | Multi-select | DHT · WebRTC · Delegation · Matching · Crypto |
| **Export logs** | Action | Download structured log JSON |
| **Peer stats** | Display | `peer.delegation.stats`, `peer.match.stats` live readout |
| **DHT inspector** | Panel | Show DHT key/value pairs in local node |
| **Force cold start** | Action | Clear DHT cache; re-announce from scratch |
| **Simulate offline** | Toggle | Disconnect from network for testing |
| **Model inspector** | Display | Loaded model ID, hash, size, load time |
| **Queue inspector** | Display | Offline action queue; count + preview |
| **Run at tier** | URL param | `?tier=low&delegate=true` (for local testing) |

---

## 13. CLI / Node.js Operator Dashboard

For operators running the Node.js server form factor (`@isc/apps/node`) as a supernode.

### 13.1 CLI Commands

```
isc embed <text>              Embed text and print JSON vector
isc match <text> <text>       Compute cosine similarity between two texts
isc announce --channel <file> Announce a channel JSON to DHT
isc query --text <text>       Query DHT for proximal peers
isc supernode start           Start serving as supernode
isc supernode status          Show delegation health metrics
isc export-key                Export keypair (encrypted) to file
isc import-key <file>         Import keypair from file
isc peer list                 List connected peers
isc dht inspect <key>         Inspect a DHT key
```

### 13.2 HTTP Monitoring Dashboard

When run as a server, exposes a monitoring UI at `http://localhost:3001/admin`:

**Overview panel**:
- Peer ID and uptime.
- Connections: active, peak, total.
- DHT: put/get rate, key count.
- Delegation: requests served (last 24h), success rate, avg latency.

**Supernode health**:
- Displays `successRate`, `avgLatencyMs`, `requestsServed24h`.
- Historical graph over 7 days.

**Prometheus metrics endpoint**: `/metrics` exports `isc_*` metric families.

**Log stream**: Live tail of structured JSON log.

---

## 14. Connectivity & Error States

These cross-cut all screens. Every error state must give the user a clear next action.

| Error | Display | User Action |
|---|---|---|
| **No bootstrap peers** | Full-screen: *"Searching for the network… No peers found yet."* | Retry · Enter custom bootstrap peer · Open seed tab |
| **NAT traversal failed** | Inline in Match Explorer: *"Can't reach [Peer] directly — trying relay…"* | Auto-retries; can cancel |
| **Model download failed** | Banner: *"Model download failed. Using word-matching mode."* | Retry · Use delegation · Continue in limited mode |
| **Signature invalid** | Peer card: ⚠️ red outline; *"This peer's announcement failed verification."* | Peer is auto-filtered; can report |
| **Rate limited (self)** | Status bar: *"Announcing too frequently — slowing down."* | Auto-resolves after window |
| **Rate limited (supernode)** | Toast: *"Supernode is busy — retrying in 60s."* | Dismiss; auto-retry |
| **DHT quota exceeded** | Status bar flash + notification | Reduce channel count; wait |
| **Offline** | Grey status bar + banner; all badges grey | Queues actions; auto-syncs on reconnect |
| **Browser storage full** | Modal: *"Storage is full. Delete old data or archived channels."* | Opens data management; clear cache |
| **Model mismatch** | Match list: *"Some peers use a different model and won't appear."* | Migration prompt if > 50% of matches use new model |
| **Key compromise** | Urgent banner: *"Your key may be compromised. Rotate now."* | Opens key rotation flow |
| **Supernode misbehavior** | Toast: *"A helper node returned bad data — it's been blocked."* | Dismissed automatically; reported |

---

## 15. Offline-First Behaviors

| Action | Offline Behavior |
|---|---|
| **Send chat message** | Queued; delivery badge shows "pending" |
| **Post a post** | Queued; shown in local feed with "pending" badge |
| **Like / repost** | Queued action synced on reconnect |
| **Follow / unfollow** | Queued; optimistic UI update |
| **Edit channel** | Applied locally; DHT re-announce queued |
| **Mute peer** | Applied locally immediately; DHT event queued |
| **Key rotation** | Not allowed offline (requires DHT announcement of old key) |
| **Delegation request** | Falls back to local minimal model immediately |

**Sync UI**:
- When reconnecting: sliding banner *"Syncing X actions…"* with progress.
- Conflict resolution for concurrent channel edits: LWW-Map logic; toast notifies *"Channel updated by another session — your changes were merged."*

---

## 16. Accessibility Requirements (WCAG 2.1 AA)

All screens must meet:

| Requirement | Implementation |
|---|---|
| **Screen readers** | All interactive elements have `aria-label` or visible text. Dynamic content uses `aria-live="polite"` / `aria-live="assertive"` for match results and chat messages. Tested with NVDA, VoiceOver, JAWS. |
| **Keyboard navigation** | Tab order follows visual layout. All functionality available without mouse. Skip links at top of each view. |
| **Focus indicators** | 3px outline, ≥ 3:1 contrast ratio. Always visible (not hidden on focus). |
| **Color contrast** | Text ≥ 4.5:1; large text ≥ 3:1; UI components ≥ 3:1. Verified with axe / WAVE. |
| **Destructive confirmations** | Delete / mute / rotate key actions require confirmation step or provide undo. |
| **Time limits** | Match expiry and chat timeouts are adjustable in Matching settings (or extendable with a warning before expiry). |
| **No mouse-only paths** | All hover-only actions have keyboard / tap equivalents. |
| **Cognitive load** | Labels are descriptive; icons always paired with text; consistent navigation patterns across all views. |

---

## 17. Internationalization

| Requirement | Notes |
|---|---|
| **Non-Latin scripts** | Embeddings work for CJK, Arabic, Cyrillic, Devanagari (model-dependent; user informed if word-hash fallback is active for their language). |
| **RTL layout** | Full UI mirroring for Arabic, Hebrew, Persian, Urdu. |
| **Date/number formatting** | User's locale respected (ISO 8601 for storage; locale-formatted for display). |
| **Plural forms** | CLDR-compliant per locale. |
| **Text expansion** | UI accommodates 30%+ text length increase (German, Finnish). |
| **Translation pipeline** | All user-visible strings in an i18n resource file; community-contributed translations. |

---

## 18. Onboarding Flows for Specific Use Cases

### 18.1 Private Community (Trusted Network, Phase 1)

A small group sharing an invite link to connect their browser-native nodes:

1. Admin creates a channel; shares QR code.
2. Members scan QR → land on onboarding → key generation → channel auto-imported.
3. Match Explorer immediately shows community members as matches.
4. Group chat auto-forms if all 3+ members are online.

### 18.2 Conference / Event Attendees

_"machine learning in Tokyo during NeurIPS 2026"_

1. Event organizer sets up a canonical channel description and publishes it as a QR code or invite link.
2. Attendees open ISC, scan the link → channel populated with `in_location: Tokyo`, `during_time: 2026-12`.
3. Match Explorer shows other attendees sorted by topical similarity.
4. Group chat auto-forms in dense clusters (e.g., reinforcement learning researchers).

### 18.3 Serendipitous Exploration

A user who wants to discover unexpected connections:

1. Enable **Chaos mode** (dial to 0.2+) in Matching settings.
2. Set spread σ = 0.3 (high fuzziness) in Channel Editor.
3. Lower similarity threshold to 0.40.
4. Explore **Relation Map** to visually see the semantic neighborhood.
5. Use **Global Explore / Trending** to find content from adjacent communities.

### 18.4 Running a Supernode (Desktop Power User)

1. App detects High-tier device; shows one-time nudge: *"Help the network — run a supernode."*
2. User enables Supernode Mode in Settings → Delegation.
3. Configure rate limits, resource limits, optional Lightning address.
4. Status bar shows **Supernode** badge.
5. Health metrics visible in Developer panel or CLI.

### 18.5 Posting & Social Participation (Phase 3)

1. User has active channels with matches.
2. Opens **Social Feed → For You**; reads proximity-ranked posts.
3. Composes a reply or new post via **Post Composer** (linked to current channel).
4. Post propagates via DHT to all peers in the semantic neighborhood.
5. Engagement events (likes, reposts) bubble back via DHT.

### 18.6 Key Backup & Recovery

**Backup**:
1. Settings → Identity & Keys → Key backup → Choose method (file / QR / social shards).
2. Store encrypted export or distribute shards to trusted peers.

**Recovery on new device**:
1. Launch fresh ISC installation.
2. Onboarding → *"Have an existing identity?"* → Import.
3. File upload, QR scan, or shard entry (requires K=3 of N=5 shard holders to be online or provide shards).
4. Passphrase prompt (if key was encrypted).
5. Channels and follows restored from export or re-queried from DHT.

### 18.7 Migrating to a New Embedding Model

1. Banner appears: *"A new embedding model is available. Upgrade for better matches."*
2. *"What changes: Higher match quality. Requires re-download (X MB). During migration, you'll see both old and new peers."*
3. **Upgrade now** → downloads new model; restarts announce loop.
4. For 90 days: dual-announcement mode (High-tier only) — peer appears in both old and new model shards.
5. After 90 days: old-model announcements expire via TTL naturally.

---

## 19. Metrics Exposed in UI

| Metric | Location | Purpose |
|---|---|---|
| **Time-to-first-match** | Status bar on first launch | Calibration; shows system is working |
| **Match precision** | Match Explorer header (optional) | Shows % of displayed matches above threshold |
| **Similarity distribution** | Settings → Developer | Histogram of all current match scores |
| **Delegation stats** | Settings → Developer / CLI | success rate, avg latency, requests served |
| **DHT query latency** | Settings → Developer | Cold and warm response times |
| **Model load time** | One-time on first load | Expectation setting |
| **Memory footprint** | Settings → Developer | Idle and active heap size |
| **Connection success rate** | Settings → Network | % of dials that succeed |
| **Battery impact** | Settings → Developer (mobile) | Estimated drain rate |

---

## 20. Phased Feature Availability

> Features are gated by Phase. The UI should show future gated features as a subtle teaser (greyed out, labeled "Coming soon") rather than hiding them entirely, to help users understand the product's trajectory.

| Feature | Phase | Tier Required |
|---|---|---|
| Single-channel semantic matching + 1:1 chat | 1 | All |
| Multi-channel management | 1 | All |
| Supernode delegation | 1 | Low/Minimal (client); High (server) |
| Rate limiting + mute/block | 1 | All |
| Relational embeddings (all 10 tags) | 2 | High/Mid |
| Reputation system | 2 | All |
| Offline-first / PWA | 2 | All |
| Model migration tooling | 2 | All |
| Posts + For You feed + Following feed | 3 | All |
| Reactions (like, repost, reply, quote) | 3 | All |
| Profiles + follow/unfollow | 3 | All |
| Communities + shared channels | 3 | All |
| Audio Spaces | 3 | High/Mid |
| Chaos mode randomization | 3 | All |
| Lightning Network tips | 3 | Opt-in |
| AT Protocol / Bluesky bridge | 4 | All |
| Native mobile apps | 4 | All |
| ZK proximity proofs | 4 | All |
| DAO governance | 4 | All |
| Enterprise / private instances | 4 | All |

---

## 21. Navigation Model

**Bottom navigation bar** (mobile / progressive web app):

| Tab | Icon | Screen |
|---|---|---|
| Home | ⬡ | Channel List |
| Explore | 🔍 | Match Explorer (last active channel) |
| Feed (Phase 3) | 📰 | Social Feed – For You |
| Communities (Phase 3) | 👥 | Community List |
| Profile | 👤 | Own Profile |

**Header actions** (present on most screens):
- **Back** chevron.
- **Notifications** bell (badge).
- **Status bar** (bottom edge): always-visible connection state indicator.

**Deep links**:
- `isc://channel/<channelID>` → Channel Editor (edit mode).
- `isc://peer/<peerID>` → Peer Profile.
- `isc://post/<postID>` → Post Detail.
- `isc://group/<roomID>` → Group Chat join.
- `isc://community/<channelID>` → Community Channel View.
- `https://isc.network/?peer=<peerID>&relay=<relayURL>` → Direct connect onboarding.

---

## 22. Data Management

| Action | Location | Behavior |
|---|---|---|
| **Export all data** | Settings → Privacy | JSON: keypair, channels, follows, mutes, chat history, drafts |
| **Delete all data** | Settings → Privacy | Wipes IndexedDB + localStorage; shows *"After deletion, you will lose your identity and all local history."* + confirmation |
| **Clear match cache** | Settings → Network | Clears L1 and L2 match caches; forces re-query |
| **Clear model cache** | Settings → Network | Frees IndexedDB model storage; triggers re-download on next launch |
| **Manage storage quota** | Settings → Privacy / Developer | Shows per-category breakdown; allows per-category deletion |
| **Archive vs. delete channel** | Channel List swipe menu | Archive: hidden, TTL expires, recoverable; Delete: permanent |

---

## 23. Trust & Safety UX

### 23.1 Mute

- **One-click mute** from any match card, peer profile, or in-chat header.
- Sends signed `MuteEvent` to DHT.
- Muted peer disappears from all match lists immediately.
- Undo available in Notification / Muted Peers list for 5 minutes.

### 23.2 Block

- **Local only** (no DHT event). Stronger than mute.
- Peer never appears in any list, cannot dial the user.
- Managed in Settings → Privacy → Blocked Peers.

### 23.3 Report

- **From**: match card, peer profile, post, chat.
- **Reason**: Spam · Harassment · Impersonation · Off-topic · Other.
- **Description**: optional free-text.
- Sends signed `ReportEvent` to DHT (Phase 2+).
- User receives: *"Report submitted. It will be reviewed by the community."*
- Auto-mute the reported peer as a local precaution (user can undo).

### 23.4 Harassment Exit

- When in-chat similarity drops below 0.55: soft prompt *"Your conversation partner has drifted. You can continue or exit."*
- **End Chat** exits gracefully; no notification to peer.
- User's thought drift is never flagged as suspicious behavior.

### 23.5 Rate Limit Transparency

When rate limits are hit:
- Status bar shows *"Slowing down — too many actions."* (not *"You are banned"*).
- Countdown timer or estimated wait shown.
- All queued actions execute after window resets.

---

*End of ISC UI Specification v2.*
