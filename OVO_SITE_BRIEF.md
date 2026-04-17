# OVO Site Brief

Single source of truth for **OvO** marketing site, brand narrative, and product storytelling. Derived from the current iOS codebase, Firebase-backed flows, and in-app copy (not from legacy `Cursorrules.md` product descriptions, which are partially outdated).

---

## 1. Product Summary

### What OvO is
OvO is an **iOS social–creative app** centered on **digital fuse-bead (“Perler”) art** that behaves like **collectible, transferable objects**: users draw on a grid, **mint** finished works into a cloud-backed library, attach a **permanent creator note**, optionally **publish** to a public gallery, **gift** minted pieces to friends, and **surface identity** on a highly designed **digital card** (badges, showcase carousel, festival-style ID layers).

### One-line definition
**“Neon-green HUD meets bead-grid craft: mint digital bead art, inscribe meaning, gift it forward, and wear it on your OvO card.”**

### Who it is for
- **Festival / scene people** who already use digital “cards” and identity (the app still carries strong **music-festival ID card** DNA: Fest ID templates, signature blocks, QR-style networking).
- **Pixel / bead hobbyists** who want a **serious editor** (reference grids, palettes, blueprint saves) plus **online collection** behavior.
- **Collectors and gifters** who care about **ownership, gifting, and display** more than anonymous scroll feeds.

### Core value
1. **Creation** that feels like a **tool** (grid, palettes, effects, blueprints), not a toy sticker pack.  
2. **Minting** turns a draft into a **durable digital object** with metadata and cloud status.  
3. **Inscription** (creator note + optional title) makes the object **emotionally and narratively real**.  
4. **Gifting + transfer records** make circulation **meaningful** (inbox, accept flow, stats).  
5. **Public Discover** + **importable public works** connect **community distribution** with **creative lineage** (import-locked reference copies cannot be re-minted by default product rules).

### How it differs from “just pixel art / bead art / social”
| Typical app | OvO (from implementation) |
|-------------|---------------------------|
| Flat gallery export | **Stateful works** (`draft` → **mint** → `owned` → optional **send**/transfer lifecycle) tied to **Firebase** SSOT (`perlers`, `transfers`, optional `giftRequests`). |
| Colors as cosmetics | **MARD palette** + **Pro “FX beads”** (`glow`, `pulse`, `sparkle`, `rainbowPulse`) stored per cell in design data. |
| Profile photo + bio | **Composable Black Card** with **badge strip**, **showcase carousel**, **stats**, and **festival-forward** chrome. |
| Endless anonymous feed | **Discover** with **featured / saved / liked** tabs, **creator strip**, **search**, and **per-work engagement** (likes, saves, import counts on `public_works`). |

---

## 2. Brand Essence

### Product temperament
**Controlled rave energy**: premium nightclub **HUD** clarity (wireframe, panels, glow discipline) rather than chaotic rainbow noise. The app reads as **confident**, **slightly exclusive**, and **maker-first**.

### Brand keywords (copy-ready)
Mint · Inscribe · Gift · Hold · Showcase · Journey · Card · Discover · Cyber beads · Archive · Neon HUD · Collectible circulation

### Visual temperament
- **Dark-first “Neon Rave”** system: **night black** `#050608`, **acid neon green** `#B6FF32`, **deep navy** panels `#050B1A`, **Klein blue** accents for empty states / tab muted states.  
- **Light mode** is a deliberate alternate: **botanical / forest green** UI (soft mint page background, deep green typography) — same brand, **gallery-day** variant.

### Emotional atmosphere
**“I made something real → I marked it → I passed it to someone → it still remembers me.”**  
Tension between **digital** (grid, FX, cloud) and **tactile craft** (Perler reference mode, “back of the piece” inscription copy).

### Worldview / spirit
**Circulation with memory**: objects are not only posted; they can be **owned**, **shown**, **counted in Journey**, and **moved between people** while **creatorUid** and **creator note** remain part of the record.

### First impression the app should give
**High-end creative tool + collectible social object** — not a children’s sticker app, not a generic “AI art” generator.

---

## 3. Core User Journey

Ordered **primary path** as implemented in SwiftUI + backend callables:

1. **Enter the app**  
   Auth gate → optional **Terms / community rules** gate → **Main tabs** (Card home, Discover, Create, Hub).

2. **Create a work**  
   **Create** hub: pick **Cyber Beads** (free-form digital), **Perler Beads** (reference grid for real-world beading), or **Image to Beads** (**Pro**: photo → pattern via MARD palette). Libraries open **Minted** inventory; **Blueprints** store editable drafts.

3. **Edit on canvas**  
   **PerlerEditorView**: pan/zoom grid, tools, undo stack, optional **reference/import** behaviors, **blueprint** save for drafts.

4. **Choose colors / palette**  
   Palette drawer: **MARD colors**, mode switches (e.g. image vs cyber palette rules), **FX bead** pool **gated to Pro**.

5. **Mint**  
   User taps mint → **MintNoteView** (“Give it a meaning”) → required **creator note** (max 150 chars, filtered) + optional **work name** → client builds **MintedPerler**, **saves draft to cloud**, uploads optional **cover**, then calls **`mintPerler` Cloud Function** to flip **`status: owned`**, **`isMinted: true`**, timestamps.

6. **Inscription / epigraph** (in-app naming)  
   Framed as text **on the back of the piece**; stored as **`creatorNote`** (permanent, treated as immutable in product logic). Badges refer to this as **“Inscription”** (e.g. Journey 01 subtitle **“First Inscription”**).

7. **Gift to someone**  
   **Hub → Send Gift** → choose minted work you **own** → confirm → backend creates **`transfers`** doc (pending) and updates **`perlers`** toward **sent** state; separate **`giftRequests`** path also exists for streamlined gift completion and **card flowStats** updates.

8. **Recipient receives**  
   Inbox lists **pending transfers**; accept moves **ownership** while preserving **creator** identity on the work. UI surfaces **received** works distinctly from **self-minted** for showcase rules.

9. **Display on personal space**  
   **My Card** home: **Black Card** render with **showcase carousel** (selected minted pieces — **own-minted** selection rules for primary showcase writer), **badge row**, name / Fest ID overlays.

10. **Re-gift / chain**  
   **Sendable** inventory: owned minted works **not** tied to a pending outbound transfer; **creator cannot receive their own gift** (server rule). Prior recipients blocked from loops (`receivedByUids` logic in functions).

11. **Collection & identity**  
   **Minted drawer**, **folders/tags/search** (**Pro**), **Journey levels**, **badge catalog**, **public profile** via Discover.

---

## 4. Core Features

Each block: **module** · **what it does** · **user meaning** · **site treatment**

### Editor / Canvas
- **What**: Full-screen **grid editor** with presets **30×30 (free)** and **40–60 (Pro)**; drawing pipeline separates **data** from **render** (`PerlerDrawPipeline` etc.).  
- **Meaning**: Serious craft surface — OvO is a **studio**, not a single-tap filter.  
- **Site**: Hero screen-split: **HUD chrome + grid**; loop micro-video of placing beads + toggling FX.

### Colors / Palette
- **What**: **MARD** bead palette JSON; editor modes distinguish **Image** (import-driven subset) vs **Cyber** (digital + FX); **Pro locks** on FX colors.  
- **Meaning**: Color is **material language** (real bead culture) extended by **digital-only FX** for Pro expression.  
- **Site**: Palette strip macro + “**from physical beads to cyber beads**” contrast card.

### Mint system
- **What**: Client save + **`mintPerler` callable** transaction: only **`draft`** by **creator**; flips mint flags; **imported public-lineage drafts** blocked from mint (`sourcePublicWorkId` guard).  
- **Meaning**: **Mint = commitment** — the work becomes a **first-class asset** in the user’s archive.  
- **Site**: Mid-scroll **“Mint it”** chapter with **status chip** animation (`draft` → `owned`).

### Inscription / Epigraph
- **What**: **MintNoteView** enforces **non-empty** note; copy explains permanence; **content filter** runs on note.  
- **Meaning**: Emotional **provenance layer** readable on the “back” of the piece.  
- **Site**: Pull-quote style, **150-char** UI mock, phrase **“Give it a meaning”**.

### Gift flow
- **What**: **GiftExchangeFlowCoordinator** phases (`choose` → `confirm` → `loading` → `success/error`); **Hub** entry **“Send Gift”**; services integrate **`perlerSendGift`**, **`giftRequests`**, inbox in **Received** lists.  
- **Meaning**: Gifting is **ceremonial** and **stateful**, not DM attachment.  
- **Site**: **Step strip** illustration + **inbox / accept** frame.

### Provenance / exchange history
- **What**: Root collections **`perlers`** + **`transfers`**; `PerlerSSOTStore` derives **sentCount**, **receivedCount**, **inboxPendingTransfers**, **receivedMintedPerlers**; perler carries **creatorUid** vs **ownerUid**.  
- **Meaning**: You can tell **who made it** vs **who holds it** — the philosophical core for a gifting brand.  
- **Site**: **Timeline** component (creator mint → send → accept → optional re-gift), subtle **not blockchain** wording unless you add chain later.

### Profile / Home Card / “Black card” / green shell
- **What**: **MyCardView** shows **OuterPosterCard + BlackCard render** (`BlackCardConfigViewModel`, `ComponentRenderer`); **light mode** uses **green card shell** tokens (`homeCardFill`, `homeCardStroke` in `ThemeTokens`). Yearly Pro gets **distinct home treatment** (gold-toned OvO Pro signifier in logic).  
- **Meaning**: Your **public-facing identity surface** — ID + trophies + rotating works.  
- **Site**: **Portrait device mock** with **card occupying most of the canvas**; show **dark vs light** pair.

### Showcase
- **What**: **`ShowcaseSelectionService`** writes **`perlerShowcaseSelectedIds`** + **carousel previews** into **`users/{uid}/card/current`**; auto-fills from **latest minted** if empty; **Free 3 slots / Pro 5** (`showcaseSlots`).  
- **Meaning**: Curated **mini-gallery on your chest** (card), not only a buried list screen.  
- **Site**: **Carousel close-up** on the card + caption “Your top pieces, always one swipe away”.

### Journey / Badges
- **What**: **Journey 01–10** (`JourneyLevel` + `JourneyRequirements`) gates on **minted count**, **active days**, **badges unlocked**; **Archive** hub tile (**Journey & Badges**) combines roadmap + badge catalog; **BadgeEngineV3** unlocks on events (mint, social, friends count, etc.).  
- **Meaning**: Long-term **identity progression**, not one-night usage.  
- **Site**: **Vertical roadmap** + **badge grid**; highlight **“First Inscription”** naming.

### Free vs Pro
- **What**: Central **`PlanAccessController`** gates: **works cap 100 vs unlimited**, **daily gifts 3 vs 99**, **showcase 3 vs 5**, **badges displayed 5 vs 10**, **folders 0 vs 10**, **large canvas & FX & collections & image-to-beads** → **Pro**; **7-day free trial** via Firestore `trialEndsAt`; **StoreKit** products `inscribe_pro_monthly`, `com.bonistudio.ovo.pro.yearly` (+ legacy annual id).  
- **Meaning**: Free is **fully real**; Pro is **depth, scale, spectacle**.  
- **Site**: **Comparison table** + **trial CTA** (human, not SKU-heavy).

### Social / identity / collection
- **What**: **Discover** (featured creators, public grid, saved/liked), **creator profiles**, **public work detail** with engagement counters, **Contacts** with **exchange metadata** (counts, locations model exists). Minted **library** and **collections** (Pro).  
- **Meaning**: Social is **wrapped around objects and card**, not infinite Stories clone.  
- **Site**: **“Discover objects & their authors”** chapter.

---

## 5. Visual & UI Style

### Primary colors (implemented tokens)
- **Neon accent green** `#B6FF32` (`NeonRaveTheme.neonGreen`).  
- **Background** `#050608` night black; **panels** deep navy `#050B1A`.  
- **Supporting neons** (accents for modules): orange, cyan, purple, lime, yellow, pink — used as **component flavor**, not rainbow clutter.  
- **Light theme**: pale mint screen `#F7FBF6`, **forest greens** for text and strokes (`ThemeTokens` light branch).

### Day / night bias
- **Default theme persistence**: **Dark** is default in `ThemeManager`.  
- **Home** dark mode keeps **neon green wash** as outer field (`homeScreenBackground`); light mode **calms** to paper-green.

### Visual language
**Neon Rave HUD**: thin stroked rectangles, **capsule** controls, **outer glow** on important cards, **Klein blue** for “empty slot” / inactive glam, **floating icon tab bar** (no text labels — pure iconography: atom / sparkles / wand / mark).

### Typography
- Global UI font: **Barlow Condensed Light Italic** (`AppFont.name = "BarlowCondensed-LightItalic"`).  
- Delivers **tall condensed techno-editorial** voice — excellent for **headlines on-site** matching app.

### Shape language
- Cards: **20–24pt** continuous corners; **mint sheet** 12pt inner fields; **settings** cards up to **22pt** in light.  
- **Shadow tokens** labeled light / medium / heavy / **glow** tied to **neon green**.

### UI keywords (for designers)
Neon HUD · Acid green · Black glass · Wireframe · Capsule CTA · Floating nav · Showcase carousel · Badge strip · Deep navy sheet · Forest-light alternate

### One-glance aesthetic
**“Tracer-green on void-black designer tool.”**

---

## 6. Information Architecture For Website

### Above the fold (first screen)
- **Mint + Inscribe** hook (one hero line + grid visual).  
- **Card home** still — prove **identity + collection** immediately.  
- **App Store** button.

### Scroll storytelling (middle)
1. **Create modes** triptych: **Cyber / Perler / Image→Beads (Pro)**.  
2. **Mint moment** + **permanent note** UI.  
3. **Gift + inbox** emotional beats.  
4. **Discover** + **public import** (explain **respect for creators** via locked re-mint on imported references).  
5. **Journey & badges** as **long-term belonging**.

### Comparison cards
- **Free vs Pro** (caps + FX + folders + gifts/day).  
- **Cyber beads vs Perler beads** (expression vs physical fidelity).  
- **Dark vs Light** aesthetic pair.

### Timeline / motion ideas
- **`draft → mint → gift → accept`** using **transfer doc** metaphor (dots, not blockchain jargon).  
- **Showcase carousel** parallax on scroll.

### Conversion block
- **7-day trial** + **Pro unlocks expression** (FX, large grid, collections, image import) — avoid low-level Firestore wording.

### FAQ / support layer
- **Is my note really permanent?** (Yes — product intent; align legal separately.)  
- **Can I remint someone else’s import?** (No — explain **import reference lock** simply.)  
- **Data / account** basics.

---

## 7. Product Differentiators

1. **Not “just a bead editor”** — **minted lifecycle** + **card showcase** + **Discover publishing** tie craft to **identity**.  
2. **Not generic social** — graph is **object-forward** (public works, transfers) with **festival-card** identity stack.  
3. **Gifting + provenance + inscription** — answers **why** send it: the object **carries words** and **remembers the maker** even after ownership moves.  
4. **Retention hooks are dignified** — **Journey** names (“Proven Journey”, “Living Archive”) signal **maturity**.  
5. **Cyber beads** as brand phrase — digital-only **FX** beads legitimize **screen-native craft** beside **physical Perler** mode.

**Best amplification on site**: **Inscribe → Mint → Gift → Display** loop in **one continuous sentence**.

---

## 8. Monetization / Plans

### Free (implemented gates)
- Standard **30×30** canvas.  
- **MARD** palette without **FX** beads.  
- Save up to **100** works.  
- Up to **10** blueprints.  
- **3** showcase slots on card.  
- **3** outbound gifts per day (anti-spam cap).  
- **5** badges visible on card.  
- **0** collection folders.

### Pro (plus **7-day trial** while `trialEndsAt` active)
- **Large canvases** 40–60.  
- **FX beads**: glow / pulse / sparkle / rainbowPulse.  
- **Unlimited** saves & blueprints (as coded).  
- **Collections** (folders + search semantics in feature enum).  
- **5 → 10** badge slots on card.  
- **5** showcase slots.  
- **Higher daily gift quota** (99).  
- **Image to Beads** generation pipeline (Pro-gated button).

### Pro expression lift
**Bigger stage + motion materials + archival depth + faster generosity.**

### Conversion copy angles (non-technical)
- **“More room to finish the idea.”** (large canvas)  
- **“Beads that breathe.”** (FX)  
- **“Archive like you mean it.”** (unlimited saves / folders)  
- **“Gift without counting.”** (daily cap removal)

### What not to oversell technically
Avoid naming **Cloud Functions**, **Firestore paths**, or **listener debounce** — users care about **reliability** and **magic**, not infra.

---

## 9. Screens / Views Inventory

**Human names → role → site priority**

| Area | View / flow | Purpose | Site priority |
|------|-------------|---------|----------------|
| Auth | `LoginView`, `RegisterView`, `TermsGateView` | Sign-in / join / community rules acceptance | Secondary (trust) |
| Shell | `MainTabView` + `NeonTabPillBar` | 4-tab shell: Card / Discover / Create / Hub | B-roll of **chrome** |
| Home | `MyCardView` + black card stack | **Primary identity** surface | **Hero** |
| Discover | `ExploreView`, `DiscoverSearchView`, `PublicWorkDetailView`, `CreatorProfileView` | Community feed, search, work detail, creator portfolio | **Hero** secondary |
| Create | `CreateHubView` | Mode chooser + libraries/blueprints entry | **Hero** |
| Editor | `PerlerEditorView`, `PaletteDrawerView`, chrome bars | Core creation | **Hero** |
| Image import | `ImageToBeadsView` | Photo → pattern (**Pro**) | **Mid** (Pro story) |
| Blueprint import | `BlueprintImportComingSoonView` | Placeholder — rebuild announced in UI | **Do not sell as live** |
| Mint | `MintNoteView` | Inscription gate | **Hero** |
| Inventory | `MintedDrawerView` | Saved / minted / organizational surfaces | Mid |
| Hub | `HubView` | Command center tiles | Secondary |
| Gifts | `ReceivedPerlersListView`, immersive gift coordinator views | Inbox / send | **Hero** (emotion) |
| Publish | `PublicWorksManagerView`, `PublicWorkOwnerDetailView` | Manage public copies | Mid |
| Settings | `SettingsHomeView`, `SettingsProUpgradeView`, profile editors | Account + subscription | Conversion + FAQ |
| Archive | `ArchiveView` (Journey & Badges) | Long-term progression | Mid |
| Contacts | `ContactsView`, QR flows | Friend graph & exchange | Secondary (unless B2B networking angle) |
| Debug / internal | Various `Debug*` / audit helpers | Engineering | **Not for public** |

---

## 10. Suggested Website Narrative

A **scroll-native storyboard** (English voice for headers; body can stay English on-site):

1. **Hero — “Wear what you make.”**  
   Full-bleed **Black Card** with **showcase** motion; subline: **Mint bead art. Inscribe the back. Carry it on your card.**

2. **Chapter 2 — “Two kinds of truth.”**  
   Split: **Cyber beads** (pure digital + FX) vs **Perler beads** (physical-faithful grid). CTA: **Start free**.

3. **Chapter 3 — “Give it a meaning.”**  
   Zoom **MintNoteView**; quote microcopy: **“This note will stay with the piece forever.”**

4. **Chapter 4 — “Mint — the line between messing around and meaning it.”**  
   Animate simple **state chip**: Draft / Minted / Owned. Clarify **mint is irreversible commitment** (emotionally, not legal lecture).

5. **Chapter 5 — “Send the object, not a screenshot.”**  
   Visualize **gift confirm → pending → accepted**; emphasize **creator signature remains**.

6. **Chapter 6 — “Be seen without becoming noise.”**  
   **Discover**: featured creators + **public work cards**; mention **likes / saves / imports** as **respect metrics**, not vanity dopamine.

7. **Chapter 7 — “Your archive has levels.”**  
   **Journey 01–10** ladder + **badge** close-ups; line: **First Inscription** as level 1 subtitle.

8. **Chapter 8 — “Pro is bigger stage + living materials.”**  
   Large grids + FX beads + collections + **Image to Beads**; trial mention.

9. **Closing — “Download OvO”**  
   **App Store** lockup + **one** community touch (only if URL exists publicly — app JSON references a **Discord Ring** *decoration id*, not an official invite link in code reviewed here).

---

## 11. Assets Needed For Website

| Asset | Why |
|-------|-----|
| **Device screenshots** | Card home (dark + light), Editor, Mint sheet, Gift success, Discover grid, Journey hero, Pro upgrade. |
| **UI mockups** | Isolated **MintNote**, **transfer timeline**, **showcase carousel** on card. |
| **App icon** | Store + header. |
| **Badge renders** | Grid of representative unlocked badges (glow versions). |
| **User card** video loop | Subtle parallax + neon rim lighting. |
| **Artwork examples** | 6–12 grids showing **FX beads** diversity. |
| **Gift flow diagram** | 3–4 step editorial illustration. |
| **Provenance timeline** | Same as gift diagram but **creator vs owner** emphasis. |
| **App Store badge** | Standard marketing asset. |
| **Community link** | Only if product supplies stable URL (Discord/Telegram/etc.); **not hard-coded** in scanned iOS sources. |

---

## 12. Reality Check

### Clearly implemented (shippable narrative)
- **Four-tab app shell** with **theme tokens** (dark neon + light forest).  
- **Perler / Cyber editor** with **palette**, **blueprints**, **canvas presets**, **mint pipeline** (save + CF mint).  
- **MintNoteView** inscription UX + **creatorNote** persistence.  
- **Cloud-backed** `perlers` + **`mintPerler`**.  
- **Transfers / gift inbox / accept** flows in client + Firebase functions (plus **`giftRequests`** completion path).  
- **Discover** public works, **likes/saved** personal lists, **search**, **creator profiles**.  
- **Showcase** selection syncing to **`card/current`**.  
- **Journey + badges** systems with visible **level naming**.  
- **OvO Pro**: **StoreKit 2**, **PlanAccessController** limits, **trial** UX screens (`ProTrialStartedView`, ended flow).  
- **Image to Beads** path for **Pro** (`generatePattern` flow present).  
- **Contacts** + **exchange metadata** models.

### Partially implemented / intentionally gated in UI
- **Settings → Notifications** section **hidden** by `FeatureFlags.notificationsSectionVisible = false` (Hub still links **NotificationsCenter** — verify product wants this exposed on site).  
- **Blueprint Import** shows **“Coming Soon”** only — **do not claim** blueprint camera import as live.  
- **Localization strings** still say photo workflow “coming soon” in one place while UI runs generation — **marketing should QA copy against latest build**.

### Planned / speculative (do **not** claim as released)
- **`FeatureFlags.v1Frozen`** exists as a **comment-only** transfer freeze flag **without call sites** — **no user-visible freeze** detected; still, do not invent roadmap wording from it.  
- **Cursorrules.md** describes older **Core Data**-centric scope; actual app is **Firebase-first** for card/perlers — **treat that doc as engineering legacy**, not marketing SSOT.  
- Any **blockchain / NFT** wording: **not found** in scanned mint/gift implementation — **do not imply on-chain provenance** unless product adds it later.

---

## Closing: The 5 messages the OvO site should stress most

1. **Minted bead art as collectible digital objects** — not disposable doodles.  
2. **Inscription (“Give it a meaning”)** — permanence + emotion on the **back of the piece**.  
3. **Gifting + ownership history** — **creator vs holder** clarity without buzzword soup.  
4. **Identity on a Black Card** — **showcase + badges + festival ID** in one portrait.  
5. **Pro expands the stage** — **bigger grids, FX beads, archives, and photo→pattern** for people who outgrow the free cap.

---

*End of brief — align public copy with this file before design handoff.*
