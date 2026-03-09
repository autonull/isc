# 🎨 ISC UI Design Specification
## *Breakthrough • Iconic • Ergonomic • Intuitive*

> **Design Philosophy**: *"Thoughts flow, interfaces recede."*  
> ISC's UI should feel like thinking out loud with the universe — not like using software.

---

## 🌌 Core Visual Metaphor: The Semantic Cosmos

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│    ✦ You are here                                  │
│         •                                          │
│      ╭─────╮                                       │
│     ╱  your  ╲     Nearby thinkers orbit           │
│    │ thought │    in semantic proximity            │
│     ╲  cloud  ╱                                   │
│      ╰─────╯                                       │
│          •  •    •                                 │
│        (proximal peers as gentle pulses)          │
│                                                     │
│  [ Channel: "AI Ethics" ]  [ Spread: ●○○○○ ]      │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### Why This Works:
| Principle | Implementation |
|-----------|---------------|
| **Iconic** | Cosmic visualization instantly communicates "semantic space" |
| **Ergonomic** | Peripheral awareness of connections without cognitive overload |
| **Intuitive** | Proximity = relevance; no mental model translation needed |
| **Breakthrough** | Replaces list-based chat UIs with spatial cognition |

---

## 🧭 Primary Interface Zones

### 1. **The Thought Canvas** (Center Stage)
```
┌─────────────────────────────────┐
│  ╭─────────────────────────╮    │
│  │                         │    │
│  │   •                     │    │
│  │      ✦ YOU              │    │
│  │   •    •                │    │
│  │                         │    │
│  │   [ Tap peer to chat ]  │    │
│  │                         │    │
│  ╰─────────────────────────╯    │
│                                 │
│  [ + New Thought ] [ Chaos ✦ ]  │
└─────────────────────────────────┘
```

**Interactions**:
- **Drag** your thought cloud to manually reposition (triggers re-embedding)
- **Pinch/zoom** to adjust semantic granularity (spread σ)
- **Tap peer** → slide-up chat panel (non-modal, preserves context)
- **Long-press empty space** → quick channel switcher
- **Chaos Mode toggle** (✦) → adds controlled randomness for serendipitous discovery

**Ergonomic Innovations**:
```css
/* Haptic feedback on match proximity */
.peer-pulse {
  animation: gentle-pulse calc(2s / similarity) ease-in-out;
}

/* Color encodes relation type, not just similarity */
.relation-in_location { stroke: #4ade80; }  /* green = place */
.relation-during_time { stroke: #60a5fa; }  /* blue = time */
.relation-with_mood   { stroke: #f472b6; }  /* pink = emotion */

/* Accessibility: motion-reduced mode */
@media (prefers-reduced-motion) {
  .peer-pulse { animation: none; opacity: 0.85; }
}
```

---

### 2. **Channel Orbital** (Bottom Dock)
```
┌─────────────────────────────────┐
│  ○ Work  ● Evening  ○ Weekend  [+]  │
│  "distributed systems"           │
└─────────────────────────────────┘
```

**Behavior**:
- Active channel **glows** with relation-color accents
- **Swipe horizontally** to switch channels (no menu navigation)
- **Tap [+]** → minimal modal: `Name + Description + [Add Relation]`
- **Long-press channel** → context menu: *Edit • Duplicate • Archive • Delete*

**Relation Tags Visual Language**:
```
[ AI Ethics ]
├─ 📍 in_location: Neo-Tokyo
├─ 🕐 during_time: 2026
└─ 💭 with_mood: reflective

Visualized as: 
[📍Neo-Tokyo] [🕐2026] [💭reflective] 
→ subtle colored underlines on channel name
```

---

### 3. **Chat Panel** (Slide-Up, Non-Modal)
```
┌─────────────────────────────────┐
│ ╭───────────────────────────╮  │
│ │ Alex • 0.87 similarity    │× │
│ │ "Also thinking about AI   │  │
│ │ copyright frameworks..."  │  │
│ ╰───────────────────────────╯  │
│                                │
│ You: What's your take on      │
│ derivative works?             │
│                                │
│ Alex: Interesting—          │
│ I see it as...              │
│                                │
│ [ Type message...      ] [↑]  │
│ [🎤] [📎] [🔒 E2E]            │
└─────────────────────────────────┘
```

**Key Ergonomic Features**:
| Feature | Benefit |
|---------|---------|
| **Non-modal slide-up** | Maintain spatial awareness of semantic cosmos |
| **Similarity badge** | Contextual relevance always visible |
| **Auto-fade inactive chats** | Reduce visual clutter; tap to restore |
| **One-tap encryption indicator** | Privacy status glanceable, not obstructive |
| **Voice/attachment buttons** | Progressive disclosure (hide on minimal tier) |

---

### 4. **Adaptive Control Surface** (Contextual & Tier-Aware)

**High Tier (Desktop)**:
```
[Left Sidebar]          [Main Canvas]          [Right Inspector]
├─ Channels            ├─ Semantic Cosmos     ├─ Match Details
├─ Follows             ├─ Chat Panel (dock)   ├─ Relation Editor
└─ Communities         │                      ├─ Embedding Preview
                       │                      └─ Network Health
```

**Mid Tier (Tablet)**:
```
[Top App Bar]
[ Channels • Matches • Social ]  ← Tab bar
[ Semantic Cosmos (full width) ]
[ Chat Panel (slide-up) ]
```

**Low/Minimal Tier (Mobile)**:
```
[ Minimal Header: ● Channel Name ]
[ Semantic Cosmos (touch-optimized) ]
[ Bottom Sheet: Matches List ]  ← Swipe up for details
[ Floating Action Button: + New Thought ]
```

**Progressive Enhancement Logic**:
```javascript
function renderControlSurface(tier) {
  switch(tier) {
    case 'high':
      return <DesktopLayout sidebar={true} inspector={true} />;
    case 'mid':
      return <TabletLayout tabs={true} inspector={false} />;
    case 'low':
      return <MobileLayout bottomSheet={true} floatingActions={true} />;
    case 'minimal':
      return <MinimalLayout textOnly={true} noAnimations={true} />;
  }
}
```

---

## 🎨 Visual Design System

### Color Palette: Semantic Hues
```css
:root {
  /* Base */
  --bg-cosmos: #0a0e17;
  --bg-panel: rgba(22, 28, 45, 0.92);
  --text-primary: #e6e9f0;
  --text-secondary: #94a3b8;
  
  /* Semantic Relations */
  --relation-location: #4ade80;  /* Green: grounded */
  --relation-time: #60a5fa;      /* Blue: flowing */
  --relation-mood: #f472b6;      /* Pink: emotional */
  --relation-domain: #a78bfa;    /* Purple: conceptual */
  --relation-causal: #fbbf24;    /* Amber: dynamic */
  
  /* Proximity Gradient */
  --proximity-close: #22d3ee;    /* Cyan: very close (0.85+) */
  --proximity-near: #818cf8;     /* Indigo: nearby (0.70-0.85) */
  --proximity-orbit: #64748b;    /* Slate: orbiting (0.55-0.70) */
  
  /* Privacy States */
  --ephemeral: rgba(148, 163, 184, 0.4);  /* Faded = expiring */
  --encrypted: #10b981;                   /* Emerald = secure */
}
```

### Typography: Clarity Over Decoration
```css
.font-thought {
  font-family: 'Inter', system-ui, -apple-system, sans-serif;
  font-weight: 400;
  line-height: 1.6;
  letter-spacing: -0.01em; /* Tighter for dense thought */
}

.font-channel {
  font-weight: 600;
  font-feature-settings: "ss01"; /* Alternate glyphs for icons */
}

.font-similarity {
  font-variant-numeric: tabular-nums;
  font-family: 'JetBrains Mono', monospace; /* Precision for metrics */
}
```

### Motion Principles: Purposeful, Not Pretty
```css
/* Thought embedding: gentle expansion */
@keyframes thought-bloom {
  0% { transform: scale(0.95); opacity: 0.7; }
  50% { transform: scale(1.05); opacity: 1; }
  100% { transform: scale(1); opacity: 1; }
}

/* Peer appearance: fade + subtle drift */
.peer-appear {
  animation: appear-drift 0.8s cubic-bezier(0.16, 1, 0.3, 1);
}

@keyframes appear-drift {
  0% { opacity: 0; transform: translate(0, 8px); }
  100% { opacity: 1; transform: translate(0, 0); }
}

/* Respect user preferences */
@media (prefers-reduced-motion: reduce) {
  * { animation-duration: 0.01ms !important; }
}
```

---

## ♿ Accessibility & Inclusivity by Design

### Core Commitments:
1. **Semantic HTML + ARIA** for screen reader navigation of cosmic space
2. **Keyboard-first**: `Tab` cycles peers; `Enter` opens chat; `Esc` closes panels
3. **Color-blind safe**: Proximity encoded via size + pulse + label, not just hue
4. **Cognitive load management**: 
   - "Focus Mode" hides all but active chat + your thought
   - "Simplify" toggle removes relational tags for minimal tier
5. **Internationalization ready**: 
   - Relation tags use universal icons + localized text
   - Embedding model supports multilingual prompts

### Example: Screen Reader Announcement
```html
<div role="region" aria-label="Semantic proximity space">
  <div 
    role="button"
    aria-label="Alex, 87% semantic similarity, thinking about AI copyright"
    tabindex="0"
    data-similarity="0.87"
  >
    <span class="visually-hidden">Peer: Alex</span>
    <div class="peer-orb" style="--sim: 0.87"></div>
  </div>
</div>
```

---

## 🔄 Breakthrough Interaction Patterns

### 1. **Thought Sculpting** (Replacing Text Input)
```
Instead of: [____________________]

Try: 
[ Tap to speak/type ] 
→ Live embedding preview updates as you type
→ Relation tags suggested contextually:
   "AI ethics" → [📍location?] [🕐timeframe?] [💭mood?]

[ ✦ Embed Thought ] button pulses when ready
```

**Why it's breakthrough**: Users *feel* their thought taking shape in semantic space before sending.

### 2. **Proximity Gestures**
| Gesture | Action | Ergonomic Benefit |
|---------|--------|------------------|
| **Two-finger pinch** | Adjust spread (σ) | Intuitive control of "how specific am I?" |
| **Swipe peer toward you** | Prioritize match | Natural "come closer" metaphor |
| **Swipe peer away** | Mute/deprioritize | Gentle rejection without confrontation |
| **Circular drag** | Chaos Mode intensity | Playful exploration of semantic randomness |

### 3. **Ephemeral Visual Language**
```
Announcements fade gradually:
[●●●●○] 80% TTL remaining → [●○○○○] 20% TTL

Chat sessions gently blur when similarity drifts <0.6:
[Clear conversation]  ← proactive, not punitive

No "delete" buttons — things expire naturally, reducing decision fatigue
```

---

## 📱 Responsive Layout Blueprint

### Desktop (High Tier)
```
┌─────────────────────────────────────────┐
│ ISC  [Channel: AI Ethics ▼]  [⚙️] [👤] │
├─────────────┬────────────────┬──────────┤
│ Channels    │                │ Match    │
│ • Work      │   SEMANTIC     │ Details  │
│ ● Evening   │   COSMOS       │ • Alex   │
│ • Weekend   │   (WebGL)      │   0.87   │
│ [+ New]     │                │ • Sam    │
│             │                │   0.81   │
│ Follows     │                │ [View All]│
│ • @alice    │                │          │
│ • @bob      │                │ Relations│
│             │                │ [📍🕐💭] │
├─────────────┴────────────────┴──────────┤
│ [ Type thought... ] [🎤] [✦ Chaos] [↑ Embed] │
└─────────────────────────────────────────┘
```

### Mobile (Low Tier)
```
┌─────────────────┐
│ ● AI Ethics  ✦ │
├─────────────────┤
│                 │
│    ✦ YOU        │
│      •          │
│   •     •       │  ← Touch-optimized orbs
│                 │
│ [↑ Matches (3)] │  ← Bottom sheet preview
├─────────────────┤
│ [💭 New Thought]│  ← Floating action button
└─────────────────┘
```

### Minimal Tier (2G/Constrained)
```
[AI Ethics]
──────────
• Alex (0.87)
• Sam (0.81)
• Taylor (0.76)
──────────
[+ New] [Find] [Chat]
──────────
> Thinking about...
[________________]
[Embed]
```
*Text-first, zero animations, <50KB initial payload*

---

## 🔐 Privacy & Trust Visual Cues

### Transparent Security Indicators:
```
[🔒 E2E Encrypted]  ← Always visible in chat header
[⏳ Expires in 24m] ← TTL countdown for announcements
[👁️ Visible to: 3 proximal peers] ← Real-time audience indicator

No "online status" — only semantic proximity matters
No read receipts — reduces social pressure
```

### Delegation Transparency (When Using Supernodes):
```
⚡ Using delegated embedding (High-tier peer)
   [✓ Signed] [✓ Verified locally] [ⓘ Learn more]

User control: [Disable delegation for this channel]
```

---

## 🚀 Implementation Roadmap (UI-Focused)

### Phase 1: Core Experience (MVP)
- [ ] Semantic Cosmos canvas (Canvas/WebGL)
- [ ] Channel orbital + basic switching
- [ ] Slide-up chat panel
- [ ] Tier-aware adaptive layout
- [ ] Thought sculpting input

### Phase 2: Relational Depth
- [ ] Visual relation tag editor
- [ ] Proximity gesture recognition
- [ ] Ephemeral fade animations
- [ ] Accessibility audit + ARIA implementation

### Phase 3: Social Layer
- [ ] Feed view toggle (Cosmos ↔ List)
- [ ] Profile cards with semantic summary
- [ ] Community channel visualization
- [ ] Chaos Mode visual effects

### Phase 4: Polish & Scale
- [ ] PWA install prompts + offline UI states
- [ ] Advanced theming (light/dark/auto)
- [ ] Performance budget enforcement per tier
- [ ] User testing iteration loop

---

## 🎯 Success Metrics for UI

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Time-to-first-match** | <15s | From app load to first peer visible |
| **Cognitive load score** | <3/5 | Post-session survey (NASA-TLX adapted) |
| **Accessibility compliance** | WCAG 2.1 AA | Automated + manual audit |
| **Tier adaptation accuracy** | >95% | Device probe vs. actual performance |
| **User retention (Day 7)** | >40% | Analytics on returning users |

---

> **Final Design Mantra**:  
> *"The best interface is the one you forget you're using.  
> In ISC, you're not navigating a UI — you're navigating thought itself."*

✨ **Next Step**: Create interactive Figma prototype focusing on the Semantic Cosmos interaction model + tier adaptation flow.

Would you like me to:
1. Generate wireframes for a specific tier/device?
2. Detail the WebGL/cosmos rendering approach?
3. Outline the accessibility implementation checklist?
4. Create a user journey map for first-time onboarding?

# 🌀 ISC UI Design Specification v2.0
## *Breakthrough • Iconic • Ergonomic • Intuitive • Complete*

> **Design Mantra**: *"Interfaces should recede. Thought should flow."*  
> ISC's UI doesn't present software — it presents *semantic space*. Users don't navigate menus; they navigate the geometry of ideas.

---

## 🗺️ Design System Overview

```
┌─────────────────────────────────────────────────────────┐
│                    ISC DESIGN PILLARS                    │
├─────────────────────────────────────────────────────────┤
│  🎯 SEMANTIC-FIRST    │ Visualize thought, not interface │
│  ♿ ADAPTIVE BY TIER   │ High→Minimal: graceful degradation│
│  🔐 PRIVACY-EMBEDDED  │ Trust cues woven into every pixel│
│  🌐 RELATION-AWARE    │ Location•Time•Mood•Domain visual │
│  ✨ SERENDIPITY-READY │ Chaos Mode + proximity gestures  │
│  ♿ ACCESSIBLE BY DESIGN│ WCAG 2.1 AA + cognitive load mgmt│
└─────────────────────────────────────────────────────────┘
```

---

## 🎨 Core Visual Language: The Semantic Cosmos

### The Thought Field (Primary Canvas)
```
┌─────────────────────────────────────────────────────┐
│                                                     │
│        ✦ YOU (σ=0.15)                              │
│             •                                       │
│        ╭─────────╮                                  │
│       ╱  "AI ethics │            ╭─────────╮        │
│      │   in Neo-Tokyo│          │ Alex    │        │
│      │   2026 • reflective"│    │ 0.87 • 📍🕐💭│        │
│       ╲           ╱            ╰─────────╯        │
│        ╰─────────╯                                  │
│              •  •                                   │
│         Sam 0.81    Taylor 0.76                    │
│                                                     │
│  [ Channel: AI Ethics ▼ ]  [ σ: ●○○○○ ]  [ ✦ Chaos ]│
│                                                     │
└─────────────────────────────────────────────────────┘
```

**Visual Encoding System**:
| Element | Visual Property | Semantic Meaning |
|---------|----------------|------------------|
| **Your thought** | Central glow + pulsing halo | Active distribution center |
| **Peer orbs** | Size ∝ similarity; color ∝ relation type | Proximity + contextual relevance |
| **Relation badges** | 📍🕐💭🔬⚡ icons with colored underlines | Bound context (location/time/mood/domain/causal) |
| **Spread indicator (σ)** | Ring segments filled | Distribution width (narrow=specific, wide=exploratory) |
| **Chaos Mode** | Gentle particle drift + orbit perturbation | Controlled randomness for serendipity |

**Interaction Model**:
```javascript
// Thought Field Gesture Map
const gestures = {
  'tap(peer)': 'openChatPanel(peer)',
  'drag(yourThought)': 'triggerReEmbedding(newPosition)',
  'pinch(canvas)': 'adjustSpread(σ)',
  'swipe(peer, toward)': 'prioritizeMatch(peer)',
  'swipe(peer, away)': 'deprioritizeMatch(peer)',
  'longPress(empty)': 'quickChannelSwitcher()',
  'twoFingerRotate()': 'toggleChaosMode(intensity)',
  'doubleTap(peer)': 'viewSemanticBreakdown(peer)'
};
```

---

## 🧭 Adaptive Interface Architecture (Tier-Aware)

### High Tier (Desktop: ≥4 cores, ≥4GB RAM)
```
┌─────────────────────────────────────────────────────┐
│ ISC  [🔍 Semantic Search]  [⚙️] [👤] [🔔]          │
├─────────────┬────────────────────────┬──────────────┤
│ 📁 CHANNELS │                        │ 🔍 INSPECTOR │
│ ├─ ● Work   │    SEMANTIC COSMOS     │ ├─ Match     │
│ ├─ ○ Evening│    (WebGL + HNSW)      │ │  Alex      │
│ ├─ ○ Weekend│                        │ │  • 0.87 sim│
│ ├─ [+ New]  │                        │ │  • 📍Neo-Tokyo│
│ │           │                        │ │  • 🕐2026   │
│ 👥 FOLLOWS  │                        │ │  • 💭reflective│
│ ├─ @alice   │                        │ ├─ Relations │
│ ├─ @bob     │                        │ │ [📍][🕐][💭]│
│ │           │                        │ ├─ Embedding │
│ 🌐 COMMUNITIES│                      │ │ [Preview]  │
│ ├─ AI Ethics│                        │ └─ Network   │
│ └─ Web3 Dev │                        │   [Health]   │
├─────────────┴────────────────────────┴──────────────┤
│ 💭 [ Type thought... ] [🎤] [📎] [✦ Chaos 30%] [↑ Embed] │
└─────────────────────────────────────────────────────┘
```

**Key Features**:
- Multi-panel layout with resizable panes
- Real-time embedding preview as user types
- Relation tag editor with autocomplete from ontology
- Supernode delegation status indicator (`⚡ Delegated` / `✓ Local`)
- Advanced inspector: embedding vectors, similarity breakdown, network topology

### Mid Tier (Tablet / Mid-range Phone)
```
┌─────────────────────────────────┐
│ [≡] ISC  [AI Ethics▼]  [⚙️][👤] │
├─────────────────────────────────┤
│ [ Semantic Cosmos (touch-optimized) ]│
│                                     │
│    ✦ YOU                           │
│      •                             │
│   •     •   •                      │
│  Alex  Sam  Taylor                │
│                                     │
│ [↑ Matches • Chats • Feed] ← Tab bar│
├─────────────────────────────────┤
│ [💭 New Thought]  [✦ Chaos]      │
└─────────────────────────────────┘
```

**Adaptations**:
- Collapsible side panels → bottom sheet navigation
- Touch-optimized orb sizing (min 44px tap targets)
- Simplified relation editor (icon picker + text input)
- Delegated computation badge (`⚡` appears only when active)

### Low Tier (Budget Phone / Slow Connection)
```
┌─────────────────┐
│ ● AI Ethics  ✦ │
├─────────────────┤
│ • Alex (0.87)  │
│ • Sam (0.81)   │ ← Ranked list fallback
│ • Taylor (0.76)│
│                 │
│ [↑ View Cosmos]│ ← Optional toggle to visual mode
├─────────────────┤
│ > Thinking about...│
│ [______________] │
│ [Embed] [🎤]    │
└─────────────────┘
```

**Progressive Degradation**:
- Default to ranked list view; visual cosmos available via toggle
- Word-hash fallback UI indicator (`#` prefix on similarity scores)
- No real-time preview; embed on submit only
- Delegated requests show clear status: `⏳ Computing...` → `✓ Matched`

### Minimal Tier (2G / Constrained)
```
[AI Ethics]
──────────
# Alex 0.87
# Sam 0.81
──────────
[+ New] [Find] [Chat]
──────────
> [Text input only]
[Embed]
──────────
[⚙️ Settings] [🔐 Privacy]
```

**Zero-Compromise Principles**:
- Text-only interface; <50KB initial payload
- No animations, no WebGL, no WASM
- All computation delegated or using word-hash fallback
- Clear delegation transparency: `Using delegated embedding (High-tier peer)`

---

## 🏷️ Relation Ontology: Visual Language System

### Icon + Color + Motion Encoding
```css
:root {
  /* Relation Icons (Unicode + Custom SVG fallback) */
  --icon-location: '📍';  /* U+1F4CD */
  --icon-time: '🕐';      /* U+1F550 */
  --icon-mood: '💭';      /* U+1F4AD */
  --icon-domain: '🔬';    /* U+1F52C */
  --icon-causal: '⚡';    /* U+26A1 */
  
  /* Semantic Colors (Color-blind safe palette) */
  --rel-location: #22c55e;  /* Green: grounded, spatial */
  --rel-time: #3b82f6;      /* Blue: flowing, temporal */
  --rel-mood: #ec4899;      /* Pink: emotional, tonal */
  --rel-domain: #8b5cf6;    /* Purple: conceptual, categorical */
  --rel-causal: #f59e0b;    /* Amber: dynamic, causal */
  
  /* Accessibility: Pattern overlays for color-blind users */
  .relation-tag::after {
    content: '';
    position: absolute;
    background-image: var(--pattern);
    opacity: 0.15;
  }
  .relation-tag[data-type="location"] { --pattern: url('#diagonal-lines'); }
  .relation-tag[data-type="time"]     { --pattern: url('#dots'); }
  .relation-tag[data-type="mood"]     { --pattern: url('#waves'); }
}
```

### Compositional Embedding Visualization
```
User Input: "AI ethics in Neo-Tokyo during 2026"

Visual Breakdown:
┌─────────────────────────────────┐
│ ROOT: "AI ethics"               │
│ [████████████████] 0.92 norm   │
│                                 │
│ + 📍 LOCATION: "Neo-Tokyo"      │
│   [██████████] weight: 1.2     │
│   → fused vector shifts ↗      │
│                                 │
│ + 🕐 TIME: "2026"               │
│   [████████] weight: 1.0       │
│   → temporal constraint applied│
│                                 │
│ + 💭 MOOD: "reflective"         │
│   [██████] weight: 0.8         │
│   → emotional tone modulates   │
│                                 │
│ FINAL: [██████████████] σ=0.15 │
│ [✦ Embed Thought] button pulses│
└─────────────────────────────────┘
```

**Why This Works**: Users *see* how their thought is being geometrically constructed — building trust in the matching process and enabling intentional refinement.

---

## 💬 Chat Experience: Context-Preserving & Ephemeral

### Slide-Up Chat Panel (Non-Modal)
```
┌─────────────────────────────────┐
│ ╭───────────────────────────╮  │
│ │ Alex • 0.87 • 📍🕐💭    │× │
│ │ "Also thinking about AI   │  │
│ │ copyright frameworks..."  │  │
│ ╰───────────────────────────╯  │
│                                │
│ [🔒 E2E Encrypted] [⏳ 24m TTL]│
│                                │
│ You: What's your take on      │
│ derivative works in ML?       │
│                                │
│ Alex: Interesting—          │
│ I see it as a spectrum...   │
│ [••• typing indicator]      │
│                                │
│ [ Type message...      ] [↑]  │
│ [🎤] [📎] [🔒] [⋮ More]      │
└─────────────────────────────────┘
```

**Ergonomic Innovations**:
| Feature | Implementation | Benefit |
|---------|---------------|---------|
| **Similarity badge** | Persistent `0.87` + relation icons | Contextual relevance always visible |
| **E2E + TTL indicators** | Subtle status bar, not modal | Privacy/ephemerality glanceable |
| **Auto-fade on drift** | Chat blurs when sim < 0.6; tap to restore | Natural conversation decay, no awkward exits |
| **One-tap encryption toggle** | `[🔒]` → `[🔓]` with confirmation | User control without friction |
| **Progressive attachment UI** | `[📎]` expands to: `[🖼️] [🔗] [📄]` | Power features hidden until needed |

### Group Chat Formation (Auto-Mesh)
```
When 3+ peers match within σ threshold:

[ System: 3 proximal thinkers detected ]
[ Join Group Chat? ] [ Stay 1:1 ]

If joined:
┌─────────────────────────────────┐
│ 👥 Group: AI Ethics • Neo-Tokyo │
│ Alex • Sam • You • (2 more)    │
│                                │
│ [💬 Chat] [🎙️ Audio] [👁️ View Only]│
│                                │
│ Alex: What if we...           │
│ Sam: +1, and also...          │
│ You: [typing...]              │
│                                │
│ [💬 Message...] [↑]           │
└─────────────────────────────────┘
```

**Design Principle**: Group formation is *opt-in*, never forced. Users retain agency over conversation scale.

---

## ⚡ Supernode Delegation: Transparent & Trustworthy

### Delegation Status Indicators
```
In Settings → Delegation:
┌─────────────────────────────────┐
│ Delegation Status               │
│                                 │
│ ✓ Using delegated embedding     │
│   Provider: @supernode-alice    │
│   Model: all-MiniLM-L6-v2      │
│   Verified: ✓ Signature ✓ Norm  │
│                                 │
│ [ Disable for this channel ]    │
│ [ View delegation logs ]        │
└─────────────────────────────────┘
```

### Real-Time Delegation Flow Visualization
```
When Low-tier user embeds thought:

[ Embedding Thought... ]
├─ 📤 Request sent to @supernode-alice
├─ 🔐 Encrypted with their public key
├─ ⏳ Computing (247ms)
├─ ✅ Received & verified locally
│  ├─ ✓ Signature valid
│  ├─ ✓ Embedding norm = 1.002
│  └─ ✓ Model version matches
└─ 🎯 Finding matches...

[✓ Matched: 3 proximal peers]
```

**Trust-Building Design**:
- Every delegated step is visible and verifiable
- Local verification results shown explicitly
- One-tap disable per channel for sensitive thoughts
- No delegation = clear fallback indicator (`#` prefix on scores)

---

## ♿ Accessibility & Inclusivity: Built In, Not Bolted On

### Core Commitments
1. **Semantic HTML + ARIA Landmarks** for screen reader navigation of cosmic space
2. **Keyboard-First Navigation**: `Tab` cycles peers; `Enter` opens chat; `Esc` closes panels; `Ctrl+K` opens channel switcher
3. **Cognitive Load Management**:
   - "Focus Mode": hides all but active chat + your thought
   - "Simplify" toggle: removes relational tags, animations, and advanced controls
   - "Text-Only" mode: disables all visual cosmos, uses ranked list
4. **Color-Blind Safe Design**: Proximity encoded via size + pulse frequency + text label, not just hue
5. **Motion Sensitivity**: Respects `prefers-reduced-motion`; all animations have non-animated fallbacks
6. **Internationalization Ready**:
   - Relation tags use universal icons + localized text
   - Embedding model supports multilingual prompts (user-selectable)
   - RTL layout support for Arabic, Hebrew, etc.

### Screen Reader Announcement Example
```html
<div role="region" aria-label="Semantic proximity space">
  <div 
    role="button"
    aria-label="Alex, 87% semantic similarity. Thinking about: AI copyright frameworks. Context: Neo-Tokyo, 2026, reflective mood."
    tabindex="0"
    data-similarity="0.87"
    data-relations="location,time,mood"
  >
    <span class="visually-hidden">Peer: Alex</span>
    <div class="peer-orb" 
         style="--sim: 0.87; --rel-color: var(--rel-location), var(--rel-time), var(--rel-mood)">
    </div>
  </div>
</div>
```

### Accessibility Testing Checklist
- [ ] All interactive elements have `aria-label` or visible text
- [ ] Color contrast ≥ 4.5:1 for text, ≥ 3:1 for UI components
- [ ] Keyboard navigation order matches visual flow
- [ ] Screen reader announces similarity scores and relation context
- [ ] Motion preferences respected across all animations
- [ ] Touch targets ≥ 44x44px on mobile tiers
- [ ] Error states announced to screen readers
- [ ] Focus indicators visible and high-contrast

---

## 🎮 Breakthrough Interaction Patterns

### 1. Thought Sculpting (Replacing Linear Text Input)
```
Traditional: [____________________] [Send]

ISC: 
[ Tap to speak/type ]
→ Live embedding preview updates as you type:
   "AI ethics" → [ROOT vector preview]
   " in Neo-Tokyo" → [📍 badge appears + vector shifts ↗]
   " during 2026" → [🕐 badge + temporal constraint applied]

[ ✦ Embed Thought ] button pulses when ready
[ 🎨 Adjust σ ] slider appears for spread refinement
```

**Why It's Breakthrough**: Users *feel* their thought taking geometric shape before sending — building intuition for semantic matching.

### 2. Proximity Gestures (Spatial Cognition)
| Gesture | Action | Ergonomic Benefit |
|---------|--------|------------------|
| **Two-finger pinch** | Adjust spread (σ) | Intuitive control of "how specific am I?" |
| **Swipe peer toward you** | Prioritize match + send subtle "hello" pulse | Natural "come closer" metaphor |
| **Swipe peer away** | Mute/deprioritize without confrontation | Gentle rejection, preserves network harmony |
| **Circular drag** | Chaos Mode intensity | Playful exploration of semantic randomness |
| **Long-press your thought** | Quick relation tag editor | Contextual editing without menu navigation |

### 3. Ephemeral Visual Language
```
Announcements fade gracefully:
[●●●●○] 80% TTL remaining → [●○○○○] 20% TTL

Chat sessions gently blur when similarity drifts <0.6:
[ Conversation fading due to thought drift ]
[ Extend? ] [ Let expire ] ← proactive, not punitive

No "delete" buttons — things expire naturally, reducing decision fatigue
```

**Psychological Benefit**: Users aren't burdened with content management; the system handles ephemerality elegantly.

---

## 🎨 Design Tokens & Implementation Specs

### Color System (Semantic Hues)
```css
:root {
  /* Base Theme (Dark Cosmos) */
  --bg-cosmos: #0a0e17;
  --bg-panel: rgba(22, 28, 45, 0.92);
  --bg-elevated: rgba(30, 38, 60, 0.96);
  --text-primary: #e6e9f0;
  --text-secondary: #94a3b8;
  --border-subtle: rgba(148, 163, 184, 0.2);
  
  /* Semantic Relations */
  --relation-location: #22c55e;  /* Green: grounded */
  --relation-time: #3b82f6;      /* Blue: flowing */
  --relation-mood: #ec4899;      /* Pink: emotional */
  --relation-domain: #8b5cf6;    /* Purple: conceptual */
  --relation-causal: #f59e0b;    /* Amber: dynamic */
  
  /* Proximity Gradient */
  --proximity-close: #22d3ee;    /* Cyan: 0.85+ */
  --proximity-near: #818cf8;     /* Indigo: 0.70-0.85 */
  --proximity-orbit: #64748b;    /* Slate: 0.55-0.70 */
  
  /* Privacy States */
  --ephemeral-fade: rgba(148, 163, 184, 0.4);
  --encrypted: #10b981;
  --delegated: #fbbf24;
  
  /* Light Theme Support */
  @media (prefers-color-scheme: light) {
    --bg-cosmos: #f8fafc;
    --bg-panel: rgba(255, 255, 255, 0.92);
    --text-primary: #1e293b;
    --text-secondary: #64748b;
  }
}
```

### Typography System
```css
.font-thought {
  font-family: 'Inter', system-ui, -apple-system, sans-serif;
  font-weight: 400;
  line-height: 1.6;
  letter-spacing: -0.01em; /* Tighter for dense thought */
}

.font-channel {
  font-weight: 600;
  font-feature-settings: "ss01"; /* Alternate glyphs for icons */
}

.font-similarity {
  font-variant-numeric: tabular-nums;
  font-family: 'JetBrains Mono', monospace; /* Precision for metrics */
}

.font-relation {
  font-size: 0.875rem;
  display: inline-flex;
  align-items: center;
  gap: 0.25rem;
}
```

### Motion System (Purposeful, Not Pretty)
```css
/* Thought embedding: gentle expansion */
@keyframes thought-bloom {
  0% { transform: scale(0.95); opacity: 0.7; }
  50% { transform: scale(1.05); opacity: 1; }
  100% { transform: scale(1); opacity: 1; }
}

/* Peer appearance: fade + subtle drift */
.peer-appear {
  animation: appear-drift 0.8s cubic-bezier(0.16, 1, 0.3, 1);
}

@keyframes appear-drift {
  0% { opacity: 0; transform: translate(0, 8px); }
  100% { opacity: 1; transform: translate(0, 0); }
}

/* Chaos Mode particle effect */
.chaos-particle {
  animation: drift var(--duration, 3s) linear infinite;
}

@keyframes drift {
  0%, 100% { transform: translate(0, 0); }
  25% { transform: translate(var(--drift-x, 10px), var(--drift-y, -5px)); }
  75% { transform: translate(calc(var(--drift-x, 10px) * -1), var(--drift-y, 5px)); }
}

/* Respect user preferences */
@media (prefers-reduced-motion: reduce) {
  * { animation-duration: 0.01ms !important; animation-iteration-count: 1 !important; }
  .chaos-particle { display: none; }
}
```

---

## 📱 Responsive Layout Blueprint (Code-Ready)

### Desktop (High Tier) - CSS Grid
```css
.isc-desktop {
  display: grid;
  grid-template-columns: 240px 1fr 320px;
  grid-template-rows: 56px 1fr 72px;
  grid-template-areas:
    "header header header"
    "sidebar canvas inspector"
    "input input input";
  height: 100vh;
}

.isc-header { grid-area: header; }
.isc-sidebar { grid-area: sidebar; overflow-y: auto; }
.isc-canvas { grid-area: canvas; position: relative; }
.isc-inspector { grid-area: inspector; overflow-y: auto; }
.isc-input { grid-area: input; }
```

### Mobile (Low Tier) - Flex + Bottom Sheet
```css
.isc-mobile {
  display: flex;
  flex-direction: column;
  height: 100vh;
}

.isc-canvas-mobile {
  flex: 1;
  position: relative;
  touch-action: pan-x pan-y pinch-zoom;
}

.isc-bottom-sheet {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  background: var(--bg-panel);
  border-radius: 16px 16px 0 0;
  transform: translateY(calc(100% - 60px)); /* Show header only */
  transition: transform 0.3s ease;
}

.isc-bottom-sheet[data-expanded="true"] {
  transform: translateY(0);
}
```

### Minimal Tier - Text-Only Fallback
```html
<!-- Minimal tier: zero CSS animations, zero WASM dependencies -->
<div class="isc-minimal">
  <header>
    <h1>● AI Ethics</h1>
    <button aria-label="Toggle chaos mode">✦</button>
  </header>
  
  <main>
    <ul class="match-list">
      <li><a href="#chat-alex"># Alex 0.87</a></li>
      <li><a href="#chat-sam"># Sam 0.81</a></li>
    </ul>
  </main>
  
  <form id="thought-form">
    <label for="thought-input" class="visually-hidden">Your thought</label>
    <textarea id="thought-input" rows="3" placeholder="Thinking about..."></textarea>
    <button type="submit">Embed</button>
  </form>
</div>
```

---

## 🔐 Privacy & Trust: Visual Cues at Every Layer

### Transparency Indicators
```
[🔒 E2E Encrypted]  ← Always visible in chat header
[⏳ Expires in 24m] ← TTL countdown for announcements  
[👁️ Visible to: 3 proximal peers] ← Real-time audience indicator
[⚡ Delegated to @alice] ← Supernode usage (with verification status)

No "online status" — only semantic proximity matters
No read receipts — reduces social pressure
No persistent profiles — you are your current thought distribution
```

### Delegation Trust Flow
```
When delegation is active:

1. Request encrypted with supernode's public key
2. Response signed by supernode + verified locally
3. User sees: ✓ Signature ✓ Norm ✓ Model match
4. One-tap disable: [ Disable delegation for this channel ]

Visual indicator: 
[⚡ Delegated • Verified] in status bar
```

### Data Sovereignty Controls
```
Settings → Privacy:
┌─────────────────────────────────┐
│ Data Control                    │
│                                 │
│ ✓ Ephemeral by default (TTL)    │
│ ✓ No persistent profile         │
│ ✓ Local-only chat history      │
│                                 │
│ [ Export all data ]             │
│ [ Delete all local data ]       │
│                                 │
│ Delegation Preferences:         │
│ ☑ Allow delegation (High-tier)  │
│ ☐ Disable for sensitive channels│
│                                 │
│ [ View delegation audit log ]   │
└─────────────────────────────────┘
```

---

## 🚀 Implementation Roadmap (UI-Focused)

### Phase 1: Core Experience (MVP) — Q1 2026
- [ ] Semantic Cosmos canvas (Canvas/WebGL with HNSW fallback)
- [ ] Channel orbital + basic switching with relation tags
- [ ] Slide-up chat panel with E2E + TTL indicators
- [ ] Tier-aware adaptive layout (High/Mid/Low/Minimal)
- [ ] Thought sculpting input with live embedding preview
- [ ] Basic accessibility: keyboard nav, screen reader labels
- [ ] Delegation status indicators + verification UI

### Phase 2: Relational Depth — Q2 2026
- [ ] Visual relation tag editor with ontology autocomplete
- [ ] Proximity gesture recognition (pinch, swipe, rotate)
- [ ] Ephemeral fade animations + natural conversation decay
- [ ] Advanced accessibility: motion preferences, cognitive load modes
- [ ] Internationalization: RTL support, localized relation labels
- [ ] Supernode delegation audit log + trust visualization

### Phase 3: Social Layer — Q3-Q4 2026
- [ ] Feed view toggle (Cosmos ↔ Ranked List ↔ Chronological)
- [ ] Profile cards with semantic summary + reputation badge
- [ ] Community channel visualization (shared distributions)
- [ ] Chaos Mode visual effects + serendipity metrics
- [ ] Audio Spaces UI (WebRTC mesh audio controls)
- [ ] Crypto tipping UI (Lightning Network integration)

### Phase 4: Polish & Scale — 2027+
- [ ] PWA install prompts + offline UI states
- [ ] Advanced theming (light/dark/auto + custom accent colors)
- [ ] Performance budget enforcement per tier (Lighthouse CI)
- [ ] User testing iteration loop + A/B testing framework
- [ ] ZK proximity proof visualization (future privacy feature)
- [ ] AT Protocol / Bluesky interop UI patterns

---

## 🎯 Success Metrics & Validation

### Quantitative Metrics
| Metric | Target | Measurement Method |
|--------|--------|-------------------|
| **Time-to-first-match** | <15s (High), <45s (Minimal) | From app load to first peer visible |
| **Cognitive load score** | <3/5 (NASA-TLX adapted) | Post-session survey (n≥100) |
| **Accessibility compliance** | WCAG 2.1 AA | Automated (axe-core) + manual audit |
| **Tier adaptation accuracy** | >95% | Device probe vs. actual performance telemetry |
| **User retention (Day 7)** | >40% | Analytics on returning users |
| **Delegation trust score** | >4.5/5 | User survey on delegation transparency |

### Qualitative Validation
- [ ] **First-time user test**: Can users create a channel, embed a thought, and match with a peer in <60 seconds?
- [ ] **Accessibility audit**: Screen reader users can navigate core flows without assistance
- [ ] **Tier degradation test**: Minimal tier provides core functionality on 2G connection
- [ ] **Privacy comprehension test**: Users understand what data is shared and when
- [ ] **Serendipity test**: Chaos Mode leads to meaningful cross-topic connections

---

## 🧪 Prototype & Testing Strategy

### Interactive Prototype (Figma)
```
Key Flows to Prototype:
1. First-time onboarding: capability probe → tier selection → channel creation
2. Thought sculpting: typing → live embedding preview → relation tagging → embed
3. Proximity navigation: pinch to adjust σ, swipe to prioritize/deprioritize peers
4. Chat initiation: tap peer → slide-up panel → E2E encrypted conversation
5. Delegation flow: Low-tier user → request → verification → match

Interactive Elements:
- Live embedding preview (simulated)
- Relation tag autocomplete
- Chaos Mode particle effects (toggle)
- Tier switcher (simulate High→Minimal)
- Accessibility toggle (reduce motion, high contrast)
```

### Usability Testing Protocol
```
Participants: 20 users (5 per device tier), diverse abilities
Tasks:
1. "Find someone thinking about AI ethics in your city"
2. "Adjust your thought to be more specific about time"
3. "Start a private chat with a proximal peer"
4. "Disable delegation for a sensitive thought"
5. "Use Chaos Mode to discover unexpected connections"

Metrics Recorded:
- Task completion time
- Error rate
- Subjective ease (1-5 scale)
- Verbal feedback on semantic metaphor comprehension
```

---

> **Final Design Mantra**:  
> *"The best interface is the one you forget you're using.  
> In ISC, you're not navigating a UI — you're navigating thought itself."*

✨ **Next Immediate Actions**:
1. [ ] Build Figma prototype for Semantic Cosmos interaction model
2. [ ] Implement tier-adaptive layout system in vanilla JS/CSS
3. [ ] Create accessibility test suite (axe-core + manual checks)
4. [ ] Draft user onboarding flow with capability probe visualization

# ISC UI — Minimal Specification v3.0
## *Elegant • Essential • Unobtrusive*

> **Principle**: *Remove everything that does not serve the thought.*  
> No metaphors. No decoration. No friction.

---

## Core Interface: Three States Only

### 1. **Compose** (Default)
```
┌─────────────────┐
│ AI Ethics       │ ← Channel (tap to switch)
├─────────────────┤
│                 │
│ Thinking about… │ ← Single input field
│                 │
│ • Alex   0.87   │ ← Live matches (update as you type)
│ • Sam    0.81   │
│ • Taylor 0.76   │
│                 │
│ [ Send ]        │ ← Disabled until input ≥3 chars
└─────────────────┘
```

**Rules**:
- Input field is the focal point. Nothing competes.
- Matches appear only when semantically meaningful (sim ≥0.55).
- No previews, no animations, no badges unless critical.
- Channel name is plain text. Tap → minimal picker (no modals).

---

### 2. **Chat** (Contextual Overlay)
```
┌─────────────────┐
│ ← Alex   0.87   │ ← Back + name + similarity (functional, not decorative)
├─────────────────┤
│                 │
│ Their message   │
│ Your reply      │
│                 │
│ [ Type…      ] ↑│ ← Input + send, nothing else
└─────────────────┘
```

**Rules**:
- Chat is a slide-up overlay, not a new screen. Preserve context.
- No avatars, no timestamps, no read receipts. Only words.
- Encryption status: subtle `•` indicator in header if E2E active.
- Auto-close after 5m inactivity or similarity drift <0.6.

---

### 3. **Channel Switcher** (Transient)
```
┌─────────────────┐
│ Channels        │
├─────────────────┤
│ • Work          │
│ ● AI Ethics     │ ← Current (simple dot, no glow)
│ • Weekend       │
│ + New           │
└─────────────────┘
```

**Rules**:
- Appears as bottom sheet (mobile) or popover (desktop).
- Dismisses on tap outside or selection.
- No descriptions, no icons. Names only.

---

## Visual Language: Restraint as Strategy

### Color
```css
:root {
  --bg: #ffffff;           /* or #0a0a0a for dark */
  --text: #1a1a1a;         /* or #e6e6e6 */
  --muted: #666666;        /* secondary text */
  --accent: #0066cc;       /* one action color, used sparingly */
  --border: #e0e0e0;       /* subtle dividers only where needed */
}
```
- No semantic color coding. Similarity is numeric; context is textual.
- Accent color used only for: active input border, send button, current channel indicator.

### Typography
```css
body {
  font-family: system-ui, -apple-system, sans-serif;
  font-size: 16px;
  line-height: 1.5;
  font-weight: 400;
}
.similarity {
  font-variant-numeric: tabular-nums;
  color: var(--muted);
  margin-left: 0.5rem;
}
```
- One font stack. No weights beyond regular/medium.
- Numbers monospaced for easy scanning.

### Layout
- **Mobile**: Single column, vertical flow.
- **Desktop**: Two columns (matches list + chat) only when screen ≥768px.
- **No sidebars, no inspectors, no panels** unless absolutely required by function.

---

## Interaction Model: Predictable, Not Clever

| Action | Result | Why |
|--------|--------|-----|
| Type in input | Matches update after 300ms debounce | Immediate feedback, no lag, no preview theater |
| Tap match | Chat overlay slides up | Direct path to connection |
| Swipe chat down | Close chat | Natural dismissal gesture |
| Long-press match | Show context: `[📍 Neo-Tokyo] [🕐 2026]` | Reveal only when requested |
| Tap channel name | Show channel list | Minimal context switching |
| Pull down on compose | Refresh matches | Standard gesture, no new paradigm |

**No gestures for**: prioritizing, muting, chaos mode, relation editing.  
*If a feature requires a gesture tutorial, it doesn't belong.*

---

## Relation Context: Textual, Not Visual

Instead of colored badges or icons:

```
Alex • 0.87
[Neo-Tokyo • 2026 • reflective]  ← Plain text, muted color, revealed on long-press
```

**Rules**:
- Relations appear only on demand (long-press or tap info icon).
- Format: `•` separated, plain text, no icons.
- Editing relations: tap `[Edit context]` in long-press menu → simple text fields.

*Rationale*: Icons require learning. Text is universal. Color requires interpretation. Numbers and words are precise.

---

## Ephemeral by Default: No Visual Noise

- No TTL countdowns. No fading animations.
- Matches simply disappear when expired or drifted.
- Chat overlay auto-closes with subtle toast: `Conversation ended`.
- No confirmation dialogs. No "are you sure?" — actions are reversible or inconsequential.

*Trust the user. Trust the system. Remove the scaffolding.*

---

## Accessibility: Invisible by Design

- Semantic HTML: `<input>`, `<ul>`, `<li>`, `<button>`.
- ARIA only where native semantics insufficient (e.g., live region for match updates).
- Focus states: high-contrast outline, no animation.
- Touch targets: 44px minimum, enforced by layout, not decoration.
- Reduced motion: respected by default (no motion to reduce).

*Accessibility is not a feature. It is the baseline.*

---

## Tier Adaptation: Graceful, Not Complex

| Tier | Adaptation |
|------|-----------|
| **High** (Desktop) | Two-column layout when beneficial; otherwise same as mobile |
| **Mid** (Tablet) | Same as mobile; optional landscape two-column |
| **Low** (Mobile) | Single column, minimal chrome |
| **Minimal** (2G) | Text-only HTML; no JS required for core read/compose flow |

**No feature flags. No UI branches.**  
Same components, same code, different layout constraints.

---

## Delegation: Silent When Possible, Clear When Needed

- No status badges during normal operation.
- If delegation fails or is disabled: simple inline message `Matches limited (local mode)`.
- Settings → Delegation: plain toggle + one-line explanation.

*Transparency without anxiety. Control without clutter.*

---

## Implementation: Less Code, More Clarity

### Component Structure (Pseudo-React)
```jsx
function ISC() {
  const [channel, setChannel] = useState('AI Ethics');
  const [query, setQuery] = useState('');
  const [matches, setMatches] = useState([]);
  const [chat, setChat] = useState(null);

  // Debounced semantic search
  useDebounce(query, 300, () => {
    if (query.length >= 3) {
      const results = semanticSearch(query, channel);
      setMatches(results.filter(m => m.similarity >= 0.55));
    }
  });

  return (
    <div className="isc">
      <Header channel={channel} onSwitch={() => showChannelPicker()} />
      
      {chat ? (
        <ChatOverlay peer={chat} onClose={() => setChat(null)} />
      ) : (
        <Compose 
          value={query}
          onChange={setQuery}
          matches={matches}
          onMatchTap={setChat}
        />
      )}
      
      <ChannelPicker 
        open={isPickerOpen} 
        channels={channels}
        onSelect={setChannel}
        onClose={hideChannelPicker}
      />
    </div>
  );
}
```

### CSS: Utility-First, No Framework Overhead
```css
.isc { display: flex; flex-direction: column; height: 100vh; }
.header { padding: 1rem; border-bottom: 1px solid var(--border); }
.input { flex: 1; padding: 1rem; border: none; font: inherit; }
.input:focus { outline: 2px solid var(--accent); }
.matches { list-style: none; padding: 0; margin: 0; }
.match { padding: 0.75rem 1rem; border-bottom: 1px solid var(--border); }
.match:active { background: rgba(0,102,204,0.05); }
.chat-overlay { 
  position: fixed; bottom: 0; left: 0; right: 0; 
  background: var(--bg); border-radius: 12px 12px 0 0;
  transform: translateY(0); transition: transform 0.2s ease;
}
.chat-overlay[closed] { transform: translateY(100%); }
```

*No CSS-in-JS. No animation libraries. No state machines for simple flows.*

---

## What We Removed (And Why)

| Removed | Reason |
|---------|--------|
| Semantic Cosmos visualization | Metaphor adds cognitive load; list is faster to scan |
| Relation icons/colors | Text is universal; color requires legend |
| Chaos Mode | Serendipity should emerge from design, not a toggle |
| Similarity badges on chat | Redundant; context is in the conversation |
| Profile cards, reputation, feeds | Scope creep; ISC is for thoughts, not personas |
| Advanced inspector panels | Power user features belong in v2, not v1 |
| Haptic feedback, particle effects | Decoration without function |
| Multi-panel desktop layout | Complexity without proportional benefit |

---

## Success Criteria: Measured in Absence

✅ User completes core flow (compose → match → chat) in <20 seconds  
✅ Zero tutorial needed; interface is self-evident  
✅ No visual element exists solely to "delight"  
✅ Accessibility audit passes with zero critical issues  
✅ Minimal tier loads in <2s on 3G, functions without JS  
✅ User testing: "I didn't notice the UI" is common feedback  

---

> **Final Principle**:  
> *Perfection is achieved not when there is nothing more to add,  
> but when there is nothing left to take away.*  
> — Antoine de Saint-Exupéry

**Next Step**: Build a functional prototype in <500 lines of vanilla JS + CSS.  
Test with 5 users. Remove anything they hesitate on.


