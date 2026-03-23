# 2by3 Words — Progress Tracker

**Last updated:** 2026-03-22
**Branch:** `main`
**Build status:** 🔄 Needs Xcode open once to resolve SQLite.swift SPM package (auto-fetched from GitHub)

---

## Current Status

| Phase | Description | Status |
|-------|-------------|--------|
| Phase 0 | Xcode project setup | ✅ Done |
| Phase 0 | TTUI design system | ✅ Done |
| Phase 1 | Data collection & vocabulary DB | 🔄 Test data only (874 words); full scrape pending |
| Phase 2 | Core app connection (DB → card UI) | 🔍 Awaiting design review |
| Phase 3 | Learning features (spaced repetition, decks) | ❌ Not started |
| Phase 4 | Test features (quiz modes) | ❌ Not started |
| Phase 5 | Monetization (AdMob, StoreKit 2) | ❌ Not started |
| Phase 6 | Premium + Apple Intelligence | ❌ Not started |
| Phase 7 | Polish & launch | ❌ Not started |

---

## Completed Work (detailed)

### Phase 0 — Xcode Project Setup

| Task | Details | Commit |
|------|---------|--------|
| Xcode project init | Bundle ID `com.sahn.twobythreewords`, iOS 26.2 deployment target, SwiftUI lifecycle | `b794244` |
| App entry point | `twobythreewordsApp.swift` with `@main`, `AppDelegate.swift` stub | `b794244` |

---

### Phase 0 — TTUI Design System

All components live in `twobythreewords/TTUI/`. Every file includes an Xcode Preview.

#### Foundation Tokens (`TTUI/Foundation/`) — commit `4104514`

| File | Contents |
|------|----------|
| `TTUIColor.swift` | 14 semantic color names (card-background, card-bookmarked, text-primary, text-secondary, text-tertiary, text-placeholder, accent-blue, accent-green, accent-orange, accent-red, tag-background, tag-text, surface, divider). Each maps to a named color asset (light + dark). |
| `TTUISpacing.swift` | xs(4) / sm(8) / md(16) / lg(24) / xl(32) / xxl(48) |
| `TTUITypography.swift` | word(32, bold) / phonetic(15, mono) / partOfSpeech(11, medium, uppercase) / definition(16) / example(15, italic) / tag(11, medium) / stat(12) / tabLabel(10) |
| `TTUIRadius.swift` | card(16) / button(8) / tag(4) / actionBar bottom corners match card |
| `TTUISize.swift` | cardWidth/Height, dotSize, actionBarHeight, tabBarHeight |
| `TTUIAnimation.swift` | flipDuration(0.4), flipCurve(.easeInOut), swipeThreshold(50pt) |

#### Assets (`Assets.xcassets/`) — commit `4104514`

14 named color sets, each with Light and Dark appearance values. Matches TTUIColor token names exactly.

#### Components (`TTUI/Components/`) — commit `4104514`

| File | What it does |
|------|-------------|
| `Card/TTUIWordCardModel.swift` | Data model powering the card. Fields: `word`, `phonetic`, `partOfSpeech`, `definition`, `example`, `tags: [String]`, `difficulty: Int`, `isBookmarked: Bool`, `meanings: [MeaningBlock]`. Includes static previews: `.preview`, `.previewBookmarked`, `.previewMultiTag`. |
| `Card/TTUIWordCardFront.swift` | Card front face. Shows: word (large bold), phonetic with 🔊 icon, part-of-speech label, difficulty dots (5 dots, colored by range: 1–3 green / 4–6 blue / 7–8 orange / 9–10 red), masked definition (blur → tap to reveal). Bookmark state changes card background to `#DDEEFF` / border `#B5D4F4`. No bookmark icon inside card. |
| `Card/TTUIWordCardBack.swift` | Card back face. Shows: part-of-speech label (uppercase, muted), definitions where `showOnCard == true` (max 2), each definition with example sentence (italic, left-border accent), synonym pills. Stats row at bottom: 👁 views / ✓ correct% / ◆ familiarity — placeholder values for now (Phase 3 will wire real SwiftData). |
| `Card/TTUIWordCard.swift` | Container that owns the 3D flip animation. Uses `TTUIFlipEffect` AnimatableModifier — no timers, frame-perfect face swap at 90°. Tap → flip. Exposes `onSwipeUp` / `onSwipeDown` closures (DragGesture, threshold 50pt). |
| `Modifiers/TTUIFlipEffect.swift` | `AnimatableModifier` that drives both `rotation3DEffect` and face visibility from a single `angle` value (0→360). Avoids the "both faces visible" artifact during mid-flip. |
| `Buttons/TTUICardActionButton.swift` | Single icon-only button. SF Symbol name + tint color + tap action. |
| `Buttons/TTUICardActionBar.swift` | Bar holding 3 buttons: **Detail** (`book.closed`), **Dialogue** (`bubble.left.and.bubble.right`), **Bookmark** (`bookmark.fill` — blue when bookmarked, gray when not). Background `#D6D2C8` (darker than card). Top corners radius 0, bottom corners match card radius. Attaches flush below card with no gap. |
| `Navigation/TTUITabBar.swift` | Custom tab bar. Tabs: Deck / Test / Stats / Settings. Blur material background, haptic on tap, SF Symbol filled state on selection. |

#### Tags display (above card, outside TTUI component)

Tags (e.g. `SAT`, `GRE`) are rendered as pill badges **above** the card in `ContentView`, not inside `TTUIWordCardFront`. Style: green background `#EAF3DE`, green text `#3B6D11`, 11pt 500-weight. Hidden when word has no tags.

---

### Data Pipeline — DB Architecture & Migration

#### 3-file DB split

The original single `english_vocabulary.db` was split into three purpose-specific files:

| File | Table | Purpose |
|------|-------|---------|
| `en_words.db` | `words` | Core word data — rarely changes |
| `en_examples.db` | `examples` | AI-generated dialogues — updated frequently |
| `en_questions.db` | `questions` | AI-generated quiz questions — updated frequently |

Separating them allows independent OTA updates per content type. Future languages use the same pattern (`ko_words.db`, etc.).

| Task | Details | Commit |
|------|---------|--------|
| `migrate_to_split_db.py` | Python script: reads `english_vocabulary.db`, writes all 3 split files. Preserves all word_ids so cross-DB joins work by convention. | `f24e25e` |
| DB migration run | 874 words / 1584 examples / 1584 questions migrated. 0 warnings. All 3 `.db` files live in `2by3Words-db/`. | `49d14ba` |

#### Meanings JSON v2 migration

The raw Free Dictionary API response stored in the `meanings` column was transformed into a structured v2 format needed by the app's card rendering.

**Old structure (v1 — raw API):**
```json
[
  {
    "partOfSpeech": "adjective",
    "definitions": [
      { "definition": "...", "examples": ["..."], "synonyms": [], "antonyms": [] }
    ]
  }
]
```

**New structure (v2):**
```json
{
  "primaryPartOfSpeech": "adjective",
  "primarySummary": "Moving around; astir.",
  "primaryPhonetic": "/əˈbɛʊt/",
  "meanings": [
    {
      "partOfSpeech": "adjective",
      "phonetic": null,
      "definitions": [
        {
          "definition": "Moving around; astir.",
          "showOnCard": true,
          "example": "After my bout with illness, it took me 6 months to be up and about again.",
          "synonyms": ["around", "active"],
          "antonyms": []
        },
        {
          "definition": "In existence; apparent.",
          "showOnCard": true,
          "example": "...",
          "synonyms": [],
          "antonyms": []
        }
      ]
    }
  ]
}
```

**Transform rules (deterministic, no API):**
- `primaryPartOfSpeech` → first part of speech in array
- `primarySummary` → first 8 words of first definition
- `primaryPhonetic` → first `isPrimary=true` US pronunciation, or first phonetic
- `meaning.phonetic` → always `null` (all fall back to primaryPhonetic; only set non-null if pronunciation differs per part of speech — will be curated in real data scrape)
- `showOnCard: true` → first 2 definitions in `primaryPartOfSpeech` only; all other parts of speech get `false`
- `example` → first item in `examples[]`, or `""`
- `synonyms` / `antonyms` → carried over as-is

| Task | Details | Commit |
|------|---------|--------|
| `migrate_meanings_v2.py` | No API calls — pure Python stdlib. Adds `migrated_v2 INTEGER DEFAULT 0` column to `words` table. Re-run safe: skips words where `migrated_v2 = 1`. Processes in batches of 50 commits. | `aeaf848` |
| v2 migration run | 874/874 words migrated, 0 errors. `en_words.db` updated in `2by3Words-db/`. | `b4fdc55` |

---

### Phase 2 — Core App Connection

#### Swift Package: SQLite.swift

| Task | Details | Commit |
|------|---------|--------|
| SQLite.swift SPM | Added `SQLite.swift 0.15.3` via SPM (`XCRemoteSwiftPackageReference` in `project.pbxproj`). Source: `https://github.com/stephencelis/SQLite.swift`. Note: a few deprecation warnings appear for iOS/tvOS 11 availability markers inside the library — harmless, ignored. | `aeaf848` |

#### DB files in Xcode

| Task | Details | Commit |
|------|---------|--------|
| DB files in Resources | `en_words.db`, `en_examples.db`, `en_questions.db` copied to `twobythreewords/Resources/`. Auto-included via `PBXFileSystemSynchronizedRootGroup` — no manual drag-drop needed. | `aeaf848` |

#### Swift Models (`Models/`)

| File | Details | Commit |
|------|---------|--------|
| `WordRecord.swift` | Maps a `words` table row to a Swift struct. Nested types: `Pronunciation` (phonetic, region, isPrimary), `Meaning` (partOfSpeech, phonetic, definitions), `WordDefinition` (definition, showOnCard, example, synonyms, antonyms). Top-level v2 fields: `primaryPartOfSpeech`, `primarySummary`, `primaryPhonetic`. `toCardModel()` computed property bridges to `TTUIWordCardModel` — picks `showOnCard=true` definitions, maps tags from JSON. | `aeaf848` |
| `ExampleRecord.swift` | Maps `examples` table row. `ExampleLine` struct: `speaker: String`, `text: String`. `lines: [ExampleLine]` decoded from JSON. | `aeaf848` |
| `QuestionRecord.swift` | Maps `questions` table row. `QuestionType` enum: `.fillInBlank`, `.definition`, `.scenario`. `QuestionDifficulty` enum: `.easy`, `.medium`, `.hard` (Comparable, so `easy < medium < hard`). | `aeaf848` |

#### DatabaseService (`Services/DatabaseService.swift`) — commit `aeaf848`

Singleton (`shared`). Opens all 3 DB files read-only from the app bundle on init. If any DB is missing, logs warning and returns empty arrays — never crashes.

| Method | Signature | Notes |
|--------|-----------|-------|
| `fetchWords` | `(limit: Int, offset: Int) -> [WordRecord]` | Paginated, ordered by `id` |
| `fetchWord` | `(id: Int) -> WordRecord?` | Single word lookup |
| `fetchWords(difficulty:)` | `(difficulty: ClosedRange<Int>) -> [WordRecord]` | Filter by difficulty range |
| `fetchWords(tag:)` | `(tag: String) -> [WordRecord]` | `LIKE '%TAG%'` match on JSON tags column |
| `fetchExamples` | `(wordId: Int) -> [ExampleRecord]` | All dialogue examples for a word |
| `fetchQuestions` | `(wordId: Int) -> [QuestionRecord]` | All quiz questions for a word |

Cross-DB joins happen in Swift (not SQL) since SQLite foreign keys don't work across separate files.

#### ViewModel (`ViewModels/WordCardViewModel.swift`) — commit `aeaf848`

`@Observable` class (Swift 5.9 Observation framework, no `ObservableObject`).

| Property/Method | Notes |
|----------------|-------|
| `words: [WordRecord]` | Loaded page buffer |
| `currentIndex: Int` | Current card position |
| `currentWord: WordRecord?` | `words[safe: currentIndex]` — nil-safe via `Collection[safe:]` extension |
| `loadWords()` | Fetches first 100 words from `DatabaseService` |
| `nextWord()` | `currentIndex += 1`; triggers `loadMore()` when within 20 of end |
| `previousWord()` | `currentIndex -= 1`, floor 0 |
| `loadMore()` | Appends next 100 words; stops silently at DB end |

#### Utilities (`Utilities/Extensions.swift`) — commit `aeaf848`

`Collection[safe: index]` subscript — returns `nil` instead of crashing on out-of-bounds.

#### ContentView (`ContentView.swift`) — commit `aeaf848`

Replaced the default SwiftUI placeholder with the real main screen:

```
VStack {
    TopBar           — "2by3 Words" title (left) + iCloud status icon (right, static)
    TagPills         — green pills above card, hidden if no tags
    TTUIWordCard     — connected to WordCardViewModel.currentWord
    TTUICardActionBar
    TTUITabBar
}
```

- Swipe up → `viewModel.nextWord()`
- Swipe down → `viewModel.previousWord()`
- Card tap → flip (handled inside `TTUIWordCard`)
- `WordCardViewModel` instantiated as `@State` in ContentView; injected into card via binding

---

### CLAUDE.md History

| Version | Changes | Commit |
|---------|---------|--------|
| v1.3 | Initial engineer role prompt + UI design decisions section | `4104514` |
| v1.5 | DB architecture refactored to 3-file split (en_words, en_examples, en_questions); cross-DB join convention documented | `—` (manual sync) |
| v1.6 | Added Review Workflow section (design + product review cycle per phase); Voice Answer Mode added to Phase 6; Git rules updated (no auto-commit/push without explicit instruction); CLAUDE.md sync to public repo documented | `—` (manual sync) |

---

## In Progress

🔍 **Awaiting design review** — Phase 2 complete.

**Next step:** Sahn takes simulator screenshots (front / back / bookmarked state) and shares with PM (Claude.ai) for design + product review. PM signs off → Phase 3 begins.

**Still missing from Phase 2 (low priority — PM to confirm):**
- TTS (AVSpeechSynthesizer) — pronunciation audio on card
- Category/deck selection screen

---

## Backlog (PM-prioritized order)

### Phase 1 — Data Collection (deferred — using test data for now)
- [ ] Download NGSL / NAWL / NDL word lists (~4,300 words total)
- [ ] Download Princeton WordNet Core (~5,000 words)
- [ ] Run `collect_data.py` to crawl Free Dictionary API for all words
- [ ] Curate difficulty ratings (1–10 scale) + exam tags (SAT/GRE/TOEFL/IELTS)
- [ ] Generate `en_examples.db` — AI dialogues per word (Claude API, haiku)
- [ ] Generate `en_questions.db` — AI quiz questions per word (fill_in_blank / definition / scenario, easy/medium/hard)
- [ ] Run `migrate_meanings_v2.py` with AI curation on real data (not mechanical transform)
- [ ] Push all 3 updated DBs to `scha00/2by3-words-db` as a GitHub Release

### Phase 2 — Core Features
- [x] SQLite.swift SPM package
- [x] DatabaseService (opens all 3 DBs read-only; never throws to UI)
- [x] WordRecord / ExampleRecord / QuestionRecord Swift models
- [x] WordRecord → TTUIWordCardModel bridge (`toCardModel()`)
- [x] WordCardViewModel (`@Observable`, paginated, auto-prefetch)
- [x] ContentView: top bar + tag pills + card + action bar + tab bar
- [x] Swipe up/down navigation (DragGesture, 50pt threshold)
- [ ] TTS integration (AVSpeechSynthesizer) — pronunciation on 🔊 tap
- [ ] Category/deck selection screen

### Phase 3 — Learning Features
- [ ] SwiftData setup: `WordInteraction`, `QuestionAttempt`, `StudySession` models
- [ ] Implicit familiarity tracking: swipe timing (<1s → +0.1, 5s+ → -0.1), example tap (-0.15)
- [ ] Wire real stats into card back stats row (views, correct%, familiarity)
- [ ] Deck system: `Deck` + `DeckFilter` SwiftData models
- [ ] Spaced repetition algorithm (familiarity score + nextReviewDate)
- [ ] Statistics view
- [ ] Search functionality
- [ ] Conversation/dialogue view (bubble UI for ExampleRecord lines)

### Phase 4 — Test Features
- [ ] Word → Definition quiz (multiple choice)
- [ ] Definition → Word quiz (multiple choice)
- [ ] Fill-in-the-blank (easy/medium/hard from `en_questions.db`)
- [ ] Scenario questions (SAT-style)
- [ ] Score tracking per test type

### Phase 5 — Monetization
- [ ] AdMob banner ads (free tier)
- [ ] StoreKit 2 IAP setup ($2.99 one-time)
- [ ] Premium feature gates
- [ ] Paywall UI

### Phase 6 — Premium + Apple Intelligence
- [ ] Home widget (WidgetKit) — *triggers TTUI → local Swift Package migration*
- [ ] Lock screen widget
- [ ] Advanced statistics (Charts framework)
- [ ] Custom themes
- [ ] iCloud sync (SwiftData + CloudKit, premium only)
- [ ] Image quiz (ImageCreator API, iPhone 15 Pro+)
- [ ] Real-time AI sentences (Foundation Models, iOS 26+)
- [ ] Voice answer mode (Speech framework + Foundation Models)

### Phase 7 — Polish & Launch
- [ ] App icon & branding
- [ ] Onboarding flow (3 screens: card preview → level selection → card)
- [ ] App Store screenshots
- [ ] App Store description + ASO keywords
- [ ] TestFlight beta
- [ ] Submit to App Store

---

## Folder Structure

```
2by3Words/                          ← private repo (scha00/2by3-words)
├── CLAUDE.md                       ✅ v1.6 — engineer role + full project guide
├── PROGRESS.md                     ✅ this file (copy; source of truth in 2by3Words-db/)
├── VocabularyData/
│   ├── scripts/
│   │   ├── collect_data.py         🔄 stub — full scrape not yet run
│   │   ├── json_to_sqlite.py       ✅ (legacy — superseded by migrate_to_split_db.py)
│   │   ├── migrate_to_split_db.py  ✅ splits single DB → 3 files
│   │   ├── migrate_meanings_v2.py  ✅ transforms meanings JSON to v2 structure
│   │   └── analyze_quality.py      🔄 stub
│   └── word-lists/                 ❌ empty — real word lists not yet downloaded
│
└── twobythreewords/                ✅ Xcode project
    └── twobythreewords/
        ├── twobythreewordsApp.swift ✅ @main entry point
        ├── ContentView.swift        ✅ main screen (top bar + tags + card + action bar + tab bar)
        ├── Assets.xcassets/         ✅ 14 TTUI color sets (light + dark) + AppIcon placeholder
        ├── TTUI/                    ✅ design system
        │   ├── TTUI.swift           ✅ index / usage docs
        │   ├── Foundation/          ✅ TTUIColor, TTUISpacing, TTUITypography, TTUIRadius, TTUISize, TTUIAnimation
        │   ├── Components/
        │   │   ├── Card/            ✅ TTUIWordCardModel, TTUIWordCardFront, TTUIWordCardBack, TTUIWordCard
        │   │   ├── Buttons/         ✅ TTUICardActionButton, TTUICardActionBar (3 buttons: Detail/Dialogue/Bookmark)
        │   │   └── Navigation/      ✅ TTUITabBar
        │   └── Modifiers/           ✅ TTUIFlipEffect (AnimatableModifier)
        ├── Models/                  ✅ WordRecord, ExampleRecord, QuestionRecord
        ├── ViewModels/              ✅ WordCardViewModel
        ├── Services/                ✅ DatabaseService
        ├── Utilities/               ✅ Extensions (Collection[safe:])
        └── Resources/               ✅ en_words.db, en_examples.db, en_questions.db

2by3Words-db/                       ← public repo (scha00/2by3-words-db)
├── CLAUDE.md                       ✅ synced from private repo
├── PROGRESS.md                     ✅ this file
├── en_words.db                     ✅ 874 words, meanings v2, migrated_v2 column
├── en_examples.db                  ✅ 1584 example dialogues
└── en_questions.db                 ✅ 1584 quiz questions
```

Legend: ✅ Complete · 🔄 In progress / partial · ❌ Not started

---

## Known Issues

- **SQLite.swift iOS/tvOS 11 deprecation warnings** — appear in SPM package source, not our code. Harmless. Will resolve when SQLite.swift releases a version dropping iOS 11 availability checks.
- **meanings v2 `primarySummary` is rough** — mechanical truncation of first definition to 8 words. Fine for test data; will be AI-curated in real data scrape (Phase 1).
- **Card back stats row shows placeholder values** (0 views, 0% correct, 0.5 familiarity) — SwiftData not wired yet. Will be real data in Phase 3.
- **Bookmark state is in-memory only** — resets on app restart. SwiftData persistence in Phase 3.

---

## Notes for PM

- **Phase 2 core wiring is done.** Real words from `en_words.db` are loading and displaying in the card via swipe navigation. `WordRecord.toCardModel()` bridges the DB layer to the TTUI design system cleanly.
- **Build requirement:** Open project in Xcode once before building — Xcode will auto-fetch `SQLite.swift 0.15.3` from GitHub via SPM.
- **meanings JSON is now v2.** The `primaryPartOfSpeech`, `primarySummary`, `primaryPhonetic`, and `showOnCard` fields are populated on all 874 words. The card back uses `showOnCard=true` to pick which definitions to display. This was a mechanical transform — AI curation happens during Phase 1 real data scrape.
- **CLAUDE.md v1.6** is synced to this public repo. PM can read full architecture at `https://raw.githubusercontent.com/scha00/2by3-words-db/main/CLAUDE.md`.
- **Still missing from Phase 2 before full sign-off:** TTS (🔊 button on card), category/deck selection screen. PM to confirm if these block Phase 3 or can be deferred.
- **Phase 3 trigger:** SwiftData setup is the first thing in Phase 3. Once `WordInteraction` is wired, the card back stats row and bookmark persistence both become real automatically.
