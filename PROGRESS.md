# 2by3 Words — Progress Tracker

**Last updated:** 2026-03-22 (meanings v2 migration)
**Branch:** `main`
**Build status:** 🔄 Needs Xcode open to resolve SQLite.swift SPM package

---

## Current Status

| Phase | Description | Status |
|-------|-------------|--------|
| Phase 0 | Xcode project setup | ✅ Done |
| Phase 0 | TTUI design system | ✅ Done |
| Phase 1 | Data collection & vocabulary DB | ❌ Not started |
| Phase 2 | Core features (card UI, TTS, navigation) | 🔍 Awaiting review |
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
| CLAUDE.md v1.5 | DB architecture refactored to 3-file split (en_words, en_examples, en_questions) | `—` |
| CLAUDE.md v1.6 | Added Review Workflow section; Voice Answer Mode to Phase 6; Git rules (no auto-commit) | `—` |
| migrate_to_split_db.py | Migrates single english_vocabulary.db → en_words.db + en_examples.db + en_questions.db | `f24e25e` |
| DB migration run | 874 words / 1584 examples / 1584 questions / 0 warnings; all 3 files in `2by3Words-db/` | `49d14ba` |
| SQLite.swift SPM | Added `SQLite.swift 0.15.3` to `project.pbxproj` (XCRemoteSwiftPackageReference) | `—` |
| DB files in Xcode | `en_words.db`, `en_examples.db`, `en_questions.db` copied to `Resources/` — auto-included via `PBXFileSystemSynchronizedRootGroup` | `—` |
| WordRecord.swift | SQLite row → Swift struct; `Pronunciation`, `Meaning`, `WordDefinition` nested types; `toCardModel()` bridge to `TTUIWordCardModel` | `—` |
| ExampleRecord.swift | `ExampleLine` (speaker/text) + `ExampleRecord` | `—` |
| QuestionRecord.swift | `QuestionType` enum (fill_in_blank/definition/scenario), `QuestionDifficulty` enum (easy/medium/hard, Comparable) | `—` |
| DatabaseService.swift | Singleton; opens all 3 DBs read-only from bundle; `fetchWords`, `fetchWord`, `fetchWords(difficulty:)`, `fetchWords(tag:)`, `fetchExamples`, `fetchQuestions`; no throws to UI | `—` |
| WordCardViewModel.swift | `@Observable`; `loadWords()`, `nextWord()`, `previousWord()`; auto-prefetch next page at -20 | `—` |
| Extensions.swift | `Collection[safe:]` subscript | `—` |
| ContentView.swift | Replaced placeholder with real main screen: top bar + TTUIWordCard (swipe up/down) + TTUICardActionBar + TTUITabBar | `—` |
| Design fix sprint | Tags above card (green pills), 3-button action bar (icons only), bookmarked = blue card, colored difficulty dots, structured card back + stats row | `—` |
| migrate_meanings_v2.py | Mechanical meanings JSON → v2 structure (primaryPartOfSpeech, primarySummary, primaryPhonetic, showOnCard); 874/874 words migrated, 0 errors; `migrated_v2` column added to words table | `—` |

---

## In Progress

🔍 **Awaiting design review** — Phase 2 complete. Take simulator screenshots (front / back / bookmarked) and share with PM for design + product review before Phase 3.

---

## Backlog (PM-prioritized order)

### Phase 1 — Data Collection
- [ ] Download NGSL / NAWL / NDL word lists
- [ ] Download Princeton WordNet Core
- [ ] Run `collect_data.py` to crawl Free Dictionary API
- [ ] Curate difficulty ratings + exam tags (SAT/GRE/TOEFL/IELTS)
- [ ] Generate conversations + questions per word
- [ ] Run `collect_data.py` + `update_db.py` to build `en_words.db`
- [ ] Generate `en_examples.db` (AI conversations per word)
- [ ] Generate `en_questions.db` (AI quiz questions per word)
- [ ] Add all 3 DB files to Xcode project resources

### Phase 2 — Core Features
- [x] DatabaseService (opens en_words.db, en_examples.db, en_questions.db; joins in Swift)
- [x] Word model bridging SQLite → TTUIWordCardModel
- [ ] Category/deck selection screen
- [x] Main screen: TTUIWordCard + swipe navigation (up/down)
- [ ] TTS integration (AVSpeechSynthesizer)
- [x] Basic navigation (ContentView → TTUITabBar)

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
│   │   ├── migrate_to_split_db.py
│   │   ├── migrate_meanings_v2.py  ✅
│   │   └── analyze_quality.py
│   ├── word-lists/
│   └── output/             ❌ Empty
└── twobythreewords/        ✅ Xcode project
    └── twobythreewords/
        ├── twobythreewordsApp.swift    ✅ App entry point
        ├── ContentView.swift           ✅ Main screen wired (card + swipe + action bar + tab bar)
        ├── Assets.xcassets/            ✅ 14 TTUI color sets + AppIcon
        └── TTUI/                       ✅ Design system
            ├── TTUI.swift              ✅ Index/docs
            ├── Foundation/             ✅ 6 token files
            ├── Components/
            │   ├── Card/               ✅ Model + Front + Back + Card
            │   ├── Buttons/            ✅ CardActionButton + CardActionBar
            │   └── Navigation/         ✅ TabBar
            └── Modifiers/              ✅ TTUIFlipEffect
        ├── Models/                     ✅ WordRecord, ExampleRecord, QuestionRecord
        ├── Services/                   ✅ DatabaseService
        ├── ViewModels/                 ✅ WordCardViewModel
        ├── Utilities/                  ✅ Extensions (safe subscript)
        └── Resources/                  ✅ en_words.db, en_examples.db, en_questions.db
```

Legend: ✅ Complete · 🔄 In progress / placeholder · ❌ Not started

---

## Known Issues

_None currently._

---

## Notes for PM

- **Phase 2 core wiring is done.** ContentView now shows real words from the database via swipe navigation. Before building, open the project in Xcode once to resolve the `SQLite.swift 0.15.3` SPM package (Xcode will fetch it from GitHub automatically on first open).
- **DB files** are in `Resources/` inside the Xcode project — `PBXFileSystemSynchronizedRootGroup` picks them up automatically, no manual Xcode drag-drop needed.
- **Still missing from Phase 2:** TTS (AVSpeechSynthesizer), category/deck selection screen. PM to confirm priority.
- **Widget target** (Phase 6) is the natural trigger to extract TTUI into a local Swift Package. Full migration path documented in CLAUDE.md.
- **CLAUDE.md** is now synced to this public repo — PM can read full architecture here.
