# 2by3 Words — Progress Tracker

**Last updated:** 2026-03-07
**Branch:** `main`
**Build status:** ✅ Builds clean (iPhone 17 simulator, iOS 26.2)

---

## Current Status

| Phase | Description | Status |
|-------|-------------|--------|
| Phase 0 | Xcode project setup | ✅ Done |
| Phase 0 | TTUI design system | ✅ Done |
| Phase 1 | Data collection & vocabulary DB | ❌ Not started |
| Phase 2 | Core features (card UI, TTS, navigation) | ❌ Not started |
| Phase 3 | Learning features (spaced repetition, decks) | ❌ Not started |
| Phase 4 | Test features (quiz modes) | ❌ Not started |
| Phase 5 | Monetization (AdMob, StoreKit 2) | ❌ Not started |
| Phase 6 | Premium + Apple Intelligence | ❌ Not started |
| Phase 7 | Polish & launch | ❌ Not started |

---

## Completed

| Task | Notes | Commit |
|------|-------|--------|
| Xcode project init | Bundle ID: `com.sahn.twobythreewords`, iOS 26.2 target | `b794244` |
| TTUI Foundation tokens | TTUIColor, TTUISpacing, TTUITypography, TTUIRadius, TTUISize, TTUIAnimation | `4104514` |
| Assets.xcassets color sets | 14 named colors, light + dark mode | `4104514` |
| TTUIWordCardModel | Data model + `.preview`, `.previewBookmarked`, `.previewMultiTag` static data | `4104514` |
| TTUIWordCardFront | Front face: word, phonetic, tags, masked definition, difficulty dots | `4104514` |
| TTUIWordCardBack | Back face: full definition + example sentences, scrollable | `4104514` |
| TTUIWordCard | 3D flip card; frame-perfect face swap via AnimatableModifier (no timers) | `4104514` |
| TTUIFlipEffect | AnimatableModifier driving both rotation + face visibility from angle | `4104514` |
| TTUICardActionButton | Single button + TTUICardActionBar (Detail · Dialogue · Progress · Bookmark) | `4104514` |
| TTUITabBar | Custom tab bar with blur material, haptic, SF Symbol fill states | `4104514` |
| CLAUDE.md v1.3 | Added UI design decisions section, engineer role prompt | `4104514` |

---

## In Progress

_Nothing in progress._

---

## Backlog (PM-prioritized order)

### Phase 1 — Data Collection
- [ ] Download NGSL / NAWL / NDL word lists
- [ ] Download Princeton WordNet Core
- [ ] Run `collect_data.py` to crawl Free Dictionary API
- [ ] Curate difficulty ratings + exam tags (SAT/GRE/TOEFL/IELTS)
- [ ] Generate conversations + questions per word
- [ ] Generate `english_vocabulary.json`
- [ ] Convert to `english_vocabulary.db` via `json_to_sqlite.py`
- [ ] Add DB to Xcode project resources

### Phase 2 — Core Features
- [ ] DatabaseService (SQLite read-only: words, conversations, questions)
- [ ] Word model bridging SQLite → TTUIWordCardModel
- [ ] Category/deck selection screen
- [ ] Main screen: TTUIWordCard + swipe navigation (up/down)
- [ ] TTS integration (AVSpeechSynthesizer)
- [ ] Basic navigation (ContentView → TTUITabBar)

### Phase 3 — Learning Features
- [ ] SwiftData setup (WordInteraction, QuestionAttempt, StudySession)
- [ ] Implicit familiarity tracking (swipe timing, tap detection)
- [ ] Deck system (Deck + DeckFilter SwiftData models)
- [ ] Spaced repetition algorithm
- [ ] Statistics view
- [ ] Search functionality
- [ ] Conversation/dialogue view

### Phase 4 — Test Features
- [ ] Word → Definition quiz
- [ ] Definition → Word quiz
- [ ] Fill-in-the-blank (easy/medium/hard)
- [ ] Score tracking per test type

### Phase 5 — Monetization
- [ ] AdMob banner ads (free tier)
- [ ] StoreKit 2 IAP setup ($2.99 one-time)
- [ ] Premium feature gates
- [ ] Paywall UI

### Phase 6 — Premium + Apple Intelligence
- [ ] Home widget (WidgetKit) — *triggers TTUI → Swift Package migration*
- [ ] Lock screen widget
- [ ] Advanced statistics (Charts framework)
- [ ] Custom themes
- [ ] iCloud sync (SwiftData + CloudKit, premium only)
- [ ] Image quiz (ImageCreator API, iPhone 15 Pro+)
- [ ] Real-time AI sentences (Foundation Models, iOS 26+)

### Phase 7 — Polish & Launch
- [ ] App icon & branding
- [ ] Onboarding flow (3 screens per UI design decisions in CLAUDE.md)
- [ ] App Store screenshots
- [ ] App Store description + ASO keywords
- [ ] TestFlight beta
- [ ] Submit to App Store

---

## Folder Structure

```
2by3Words/
├── CLAUDE.md               ✅ Engineer role + full project guide
├── PROGRESS.md             ✅ This file
├── PROJECT_SUMMARY.md      ✅ High-level summary
├── VocabularyData/         ❌ Scripts exist, no DB generated yet
│   ├── scripts/
│   │   ├── collect_data.py
│   │   ├── json_to_sqlite.py
│   │   └── analyze_quality.py
│   ├── word-lists/
│   └── output/             ❌ Empty
└── twobythreewords/        ✅ Xcode project
    └── twobythreewords/
        ├── twobythreewordsApp.swift    ✅ App entry point
        ├── ContentView.swift           🔄 Default placeholder (not yet wired)
        ├── Assets.xcassets/            ✅ 14 TTUI color sets + AppIcon
        └── TTUI/                       ✅ Design system
            ├── TTUI.swift              ✅ Index/docs
            ├── Foundation/             ✅ 6 token files
            ├── Components/
            │   ├── Card/               ✅ Model + Front + Back + Card
            │   ├── Buttons/            ✅ CardActionButton + CardActionBar
            │   └── Navigation/         ✅ TabBar
            └── Modifiers/              ✅ TTUIFlipEffect
```

Legend: ✅ Complete · 🔄 In progress / placeholder · ❌ Not started

---

## Known Issues

_None currently._

---

## Notes for PM

- **ContentView.swift** is the default Xcode template — not yet wired to TTUI. Phase 2 will replace it.
- **TTUI files are not yet added to the Xcode project navigator** — they live on disk but the `.xcodeproj` file doesn't reference them (no group entries). Must be added manually in Xcode (drag folder into project navigator, "Create groups") before they compile. Build currently succeeds because Xcode's default "compile all Swift files in the target directory" setting picks them up — but for correctness they should be formally added.
- **Widget target** (Phase 6) is the natural trigger to extract TTUI into a local Swift Package. Full migration path documented in CLAUDE.md.
- **DB file** (`.db`) is gitignored. When Phase 1 is complete, the DB will be added to Xcode resources but not tracked in git per `.gitignore` rules.
