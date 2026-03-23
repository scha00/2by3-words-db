# Engineer Role — Claude Code

You are the **Engineer** for the 2by3 Words iOS app. The PM is a separate Claude instance (claude.ai) that defines tasks and priorities. Your job is to implement, not to define scope.

## Your responsibilities
1. Implement tasks as defined by the PM
2. Keep `PROGRESS.md` up to date after every meaningful change
3. Flag blockers in `PROGRESS.md` under "Blockers" and "Notes for PM"
4. Follow the project architecture defined below — don't invent new patterns

## Git rules
- **Never commit or push without explicit instruction.** Always wait for the user to say "커밋해" or "push해" before running any `git commit` or `git push` command.
- Prepare changes (stage, write commit message draft if helpful), but stop there until told to proceed.

## PROGRESS.md update rules
Update `PROGRESS.md` at the end of every session or after any meaningful change.

**What to update:**
- Move completed tasks from "In Progress" → "Completed" table
- Add new tasks to "In Progress" if you started them
- Update the folder structure legend (✅ 🔄 ❌)
- Add any blockers or decisions to "Notes for PM"
- Always update "Last updated" timestamp

**What NOT to do:**
- Don't rewrite the whole file from scratch — edit only what changed
- Don't add tasks to the backlog without PM direction
- Don't remove items from "Known Issues" unless the bug is confirmed fixed

## Before starting any task
1. Read this file for architecture and philosophy
2. Read `PROGRESS.md` for current state
3. Confirm you understand the task scope — implement exactly what was asked, no more

## Coding rules
- SwiftUI only (no UIKit)
- Offline-first — never assume network
- Keep it simple — no over-engineering
- Comments explain "why", not "what"
- Follow existing file/folder structure below

## End of session format
End every session response with:
```
## Session summary
- ✅ Completed: [what you finished]
- 🔄 In progress: [what's partially done, or "None"]
- ❌ Blocked: [any blockers, or "None"]
- 📝 PROGRESS.md: updated
```

---

# 2by3 Words - Project Guide

## Project Overview

**App Name:** 2by3 Words  
**Subtitle:** Vocabulary Cards  
**Tagline:** "Why pay subscription for word cards?"  
**Platform:** iOS (SwiftUI)  
**Pricing:** Free (with ads) + $2.99 Premium (one-time)

### Core Philosophy
- **Goal-oriented learning**: Users study vocabulary when they have specific goals (SAT, GRE, TOEFL, etc.)
- **No subscription**: Pay once, own forever. Anti-Vocabulary.com
- **Simplicity**: No over-engineering. Focus on core features only.
- **Focused**: Target specific vocabulary goals, not daily habits

### Inspiration
Named after 2×3 inch index cards used for vocabulary study. Brings the simplicity of physical flashcards to digital.

---

## Technical Stack

### iOS App
- **Language:** Swift
- **UI Framework:** SwiftUI (iOS 16+)
- **Database:** SQLite (local)
- **Text-to-Speech:** AVSpeechSynthesizer (built-in, works on all devices, no Apple Intelligence required)
- **Analytics:** CloudKit (optional, for data updates)
- **Monetization:** AdMob (free tier) + StoreKit 2 (premium)

### Apple Intelligence Features (iPhone 15 Pro+ only)
- **Foundation Models Framework** (iOS 26+): On-device LLM for real-time example sentence generation
- **ImageCreator API** (iOS 18.4+): On-device image generation for elementary visual quizzes
- Both are free, offline, privacy-preserving
- Graceful fallback to pre-generated content for non-Apple Intelligence devices

### Data Management
- **Storage:** Three local SQLite databases bundled with app per language (`en_words.db`, `en_examples.db`, `en_questions.db`)
- **User Data:** SwiftData (local, on-device)
- **Source of truth:** SQLite DBs are built and maintained directly via Python scripts — no JSON intermediary
- **Updates:** GitHub Releases via `scha00/2by3-words-db` (app checks latest release tag on launch, downloads silently, applies on next launch)

---

## Project Structure

```
2by3Words/
├── twobythreewords/              # Xcode project
│   ├── twobythreewords.xcodeproj
│   ├── twobythreewords/
│   │   ├── App/
│   │   │   ├── twobythreewordsApp.swift
│   │   │   └── AppDelegate.swift
│   │   │
│   │   ├── Models/
│   │   │   ├── Word.swift
│   │   │   ├── VocabularyDatabase.swift
│   │   │   └── UserProgress.swift
│   │   │
│   │   ├── Views/
│   │   │   ├── WordCard/
│   │   │   │   ├── WordCardView.swift
│   │   │   │   └── CardDetailView.swift
│   │   │   ├── CategorySelection/
│   │   │   │   └── CategorySelectionView.swift
│   │   │   ├── Progress/
│   │   │   │   └── ProgressView.swift
│   │   │   └── Settings/
│   │   │       └── SettingsView.swift
│   │   │
│   │   ├── ViewModels/
│   │   │   ├── WordCardViewModel.swift
│   │   │   └── ProgressViewModel.swift
│   │   │
│   │   ├── Services/
│   │   │   ├── DatabaseService.swift
│   │   │   ├── TTSService.swift
│   │   │   ├── SpacedRepetitionService.swift
│   │   │   └── AdService.swift
│   │   │
│   │   ├── Resources/
│   │   │   ├── Assets.xcassets/
│   │   │   ├── en_words.db
│   │   │   ├── en_examples.db
│   │   │   └── en_questions.db
│   │   │
│   │   └── Utilities/
│   │       ├── Constants.swift
│   │       └── Extensions.swift
│   │
│   ├── twobythreewordsWidget/    # Home/Lock Screen widgets
│   │   └── WordWidget.swift
│   │
│   └── twobythreewordsWatch/     # Apple Watch (optional)
│       └── WatchApp.swift
│
└── VocabularyData/               # Data management (outside Xcode)
    ├── word-lists/
    │   ├── english/
    │   │   ├── ngsl_words.txt        # New General Service List (CC BY-SA 4.0)
    │   │   ├── nawl_words.txt        # New Academic Word List (CC BY-SA 4.0)
    │   │   ├── ndl_words.txt         # New Dolch List (CC BY-SA 4.0)
    │   │   ├── gre_words.txt         # Curated GRE high-frequency
    │   │   └── core_wordnet.txt      # Princeton WordNet Core (BSD license)
    │   ├── korean/
    │   │   └── topik_words.txt
    │   └── spanish/
    │       └── dele_words.txt
    │
    └── scripts/
        ├── collect_data.py          # Crawl from Free Dictionary API, write directly to DB
        ├── update_db.py             # Add/update words directly in SQLite
        └── analyze_quality.py       # Check data completeness

Note: DB output files live in the public repo `scha00/2by3-words-db`, not here.
```

---

## Data Structure

### Word Categories & Tags

Tags represent **high-frequency test words** only. A tag means "this word frequently appears on this exam."

- **No tag**: General vocabulary (NGSL/NAWL/WordNet Core)
- **`SAT`**: High-frequency SAT words
- **`GRE`**: High-frequency GRE words (harder than SAT)
- **`TOEFL`**: High-frequency TOEFL words
- **`IELTS`**: High-frequency IELTS words

Difficulty and tags are **independent**:
- difficulty 7-10 does NOT automatically mean GRE tag
- A word can be difficulty 8 with no tag (just a hard general word)
- A word can be difficulty 6 with SAT tag (frequently tested but not hardest)

### Difficulty Scale

| Difficulty | Level | Source |
|-----------|-------|--------|
| 1-2 | Elementary (Grades 1-3) | NDL / Dolch / Fry |
| 3-4 | Elementary (Grades 4-6) | NGSL high-frequency |
| 5-6 | Middle School / High School | NGSL + NAWL |
| 7-8 | SAT / TOEFL / IELTS level | Curated |
| 9-10 | GRE / Advanced | Curated |

### Word Sources
- **NGSL** (New General Service List): 2,809 words — CC BY-SA 4.0
- **NAWL** (New Academic Word List): 957 words — CC BY-SA 4.0
- **NDL** (New Dolch List): ~600 words — CC BY-SA 4.0
- **Princeton WordNet Core**: ~5,000 words — BSD license
- **Free Dictionary API**: definitions, pronunciations, examples — CC BY-SA 3.0
- **Claude-curated**: difficulty ratings, tags, conversations, questions

### Attribution Required
```
"Vocabulary data from New General Service List Project (CC BY-SA 4.0)"
"Dictionary definitions from Free Dictionary API (CC BY-SA 3.0)"
"WordNet Core from Princeton University (WordNet License)"
```

---

## Database Architecture

### Why 3 separate DB files?
- `en_words.db` — core word data, rarely changes
- `en_examples.db` — AI-generated conversations, updated frequently
- `en_questions.db` — AI-generated quiz questions, updated frequently

Separating them allows independent updates per content type. When Korean/Spanish is added: `ko_words.db`, `ko_examples.db`, `ko_questions.db` etc.

### en_words.db

```sql
CREATE TABLE words (
    id                   INTEGER PRIMARY KEY AUTOINCREMENT,
    word                 TEXT UNIQUE NOT NULL,
    difficulty           INTEGER,
    tags                 TEXT,          -- JSON: ["GRE", "SAT"]
    pronunciations       TEXT,          -- JSON array
    meanings             TEXT,          -- JSON array
    sources              TEXT,          -- JSON array
    added_version        TEXT,
    last_updated_version TEXT
);

CREATE INDEX idx_word       ON words(word);
CREATE INDEX idx_difficulty ON words(difficulty);
CREATE INDEX idx_tags       ON words(tags);

CREATE TABLE metadata (
    key   TEXT PRIMARY KEY,
    value TEXT
);
```

### en_examples.db

```sql
CREATE TABLE examples (
    id      INTEGER PRIMARY KEY AUTOINCREMENT,
    word_id INTEGER NOT NULL,   -- maps to words.id in en_words.db
    lines   TEXT NOT NULL       -- JSON: [{speaker, text}, ...]
);

CREATE INDEX idx_examples_word ON examples(word_id);

CREATE TABLE metadata (
    key   TEXT PRIMARY KEY,
    value TEXT
);
```

### en_questions.db

```sql
CREATE TABLE questions (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    word_id     INTEGER NOT NULL,   -- maps to words.id in en_words.db
    type        TEXT NOT NULL,      -- "fill_in_blank" / "definition" / "scenario"
    question    TEXT NOT NULL,
    options     TEXT NOT NULL,      -- JSON: ["A", "B", "C", "D"]
    answer      TEXT NOT NULL,
    difficulty  TEXT NOT NULL,      -- "easy" / "medium" / "hard"
    explanation TEXT NOT NULL
);

CREATE INDEX idx_questions_word       ON questions(word_id);
CREATE INDEX idx_questions_type       ON questions(type);
CREATE INDEX idx_questions_difficulty ON questions(difficulty);

CREATE TABLE metadata (
    key   TEXT PRIMARY KEY,
    value TEXT
);
```

### Cross-DB word_id convention
SQLite foreign keys don't work across separate DB files. `word_id` in examples and questions maps to `words.id` in `en_words.db` by convention. The app opens all 3 DBs simultaneously and joins in Swift code, not SQL.

#### Question Difficulty Levels
| Level | Description |
|-------|-------------|
| easy | Clear context clues; wrong answers obviously wrong |
| medium | Slightly challenging; distractors are definitive but plausible |
| hard | GRE-style "best answer"; explanation required to distinguish options |

### SwiftData Models (User Data — on-device, writable)

```swift
// Tracks per-word familiarity and review schedule
@Model class WordInteraction {
    var wordId: Int
    var isBookmarked: Bool
    var familiarity: Double      // 0.0–1.0, starts at 0.5
    var lastSeenAt: Date?
    var needsReview: Bool
    var nextReviewDate: Date?
}

// Familiarity adjustment logic:
// Quick swipe (<1s):           +0.1
// Long pause (5s+):            -0.1
// Example/conversation tap:   -0.15
// Question correct:            +0.2
// Question wrong:              -0.2

// Records each quiz attempt for statistics
@Model class QuestionAttempt {
    var questionId: Int
    var wordId: Int
    var answeredAt: Date
    var isCorrect: Bool
    var selectedAnswer: String
}

// Records study sessions for analytics
@Model class StudySession {
    var startedAt: Date
    var endedAt: Date?
    var category: String
    var wordsSeenCount: Int
}
```

### Deck System (SwiftData)

```swift
@Model class Deck {
    var name: String
    var createdAt: Date
    var filters: [DeckFilter]    // tag-based or difficulty-based filters
    var includedWordIds: [Int]   // manually added words
    var excludedWordIds: [Int]   // manually excluded words
    // Effective word list = union(filter results, includedWordIds) - excludedWordIds
}

@Model class DeckFilter {
    var type: FilterType         // .tag or .difficulty
    var value: String            // e.g. "SAT", "7-9"
}
```

---

## Feature List

### Implicit Behavior Tracking (No "Know/Don't Know" Buttons)
- **Quick swipe (<1s):** familiar word → familiarity +0.1
- **Long pause (5s+):** unfamiliar → familiarity -0.1
- **Tap example or conversation:** unfamiliar → familiarity -0.15
- **Card flip:** unfamiliar (signals uncertainty)
- **Bookmark:** tap top-right corner icon (Option A — blue card state)

### Test Modes (All Devices)
- ✅ Word → Definition (multiple choice)
- ✅ Definition → Word (multiple choice)
- ✅ Fill-in-the-blank (questions table, easy/medium/hard difficulty)
- ✅ Scenario questions (SAT-style "best answer" format)
- ✅ Conversation reading (dialogues showing word in context)

### Test Modes (Apple Intelligence — iPhone 15 Pro+)
- ✅ Image quiz for elementary words (ImageCreator API, iOS 18.4+)
- ✅ Real-time AI sentence generation (Foundation Models, iOS 26+)

### Question Difficulty Strategy
- `easy`: clear context clues; wrong answers obviously wrong (antonyms)
- `medium`: plausible distractors but definitively wrong
- `hard`: GRE-style "best answer"; explanation always shown after answer
- explanation field is required for all questions (shown post-answer)

### MVP (Free Version)
- ✅ Browse all words (12,000+)
- ✅ Filter by category (GRE, SAT, TOEFL, IELTS, Elementary, Middle School)
- ✅ Filter by difficulty (1-10)
- ✅ Flashcard view (swipe to next)
- ✅ Audio pronunciation (AVSpeechSynthesizer — all devices)
- ✅ Conversation examples (dialogue-based context learning)
- ✅ Fill-in-the-blank test
- ✅ Spaced repetition algorithm
- ✅ Progress tracking
- ✅ Search words
- ✅ Custom word lists
- ✅ Banner ads (bottom)

### Premium ($2.99 one-time)
- ✅ Remove ads
- ✅ Home widget (Today's word, Review queue)
- ✅ Lock screen widget
- ✅ Advanced statistics & graphs
- ✅ Custom themes
- ✅ iCloud sync (progress across devices)
- ✅ Export word lists (PDF)
- ✅ Apple Watch app (optional)
- ✅ Image quiz (Apple Intelligence devices)

### Future (Multi-language)
- 🔲 Korean vocabulary (korean_vocabulary.db)
- 🔲 Spanish vocabulary
- 🔲 Japanese vocabulary
- 🔲 Language switcher in settings

---

## Development Phases

### Phase 1: Data Collection (Week 1)
1. Download NGSL / NAWL / NDL word lists
2. Download Princeton WordNet Core
3. Run `collect_data.py` to crawl Free Dictionary API
4. Claude curates: difficulty ratings + tags (SAT/GRE/TOEFL/IELTS HF)
5. Claude generates: conversations (separate table) + questions (fill_in_blank, definition, scenario per word)
6. Generate `english_vocabulary.json` (words, conversations, questions as top-level arrays)
7. Convert to `english_vocabulary.db` (five tables: words, conversations, questions, metadata)
8. Add to Xcode project

### Phase 2: Core Features (Weeks 2-3)
1. Xcode project setup
2. DatabaseService (SQLite read-only: words, conversations, questions)
3. Word model & view model
4. Category selection screen
5. Flashcard view (card UI, swipe gesture with timing for implicit tracking)
6. TTS integration
7. Basic navigation

### Phase 3: Learning Features (Week 4)
1. SwiftData setup: WordInteraction, QuestionAttempt, StudySession
2. Implicit familiarity tracking (swipe timing, tap detection)
3. Deck system (Deck + DeckFilter SwiftData models)
4. Spaced repetition algorithm (driven by familiarity score + nextReviewDate)
5. Statistics view
6. Search functionality
7. Conversation view (dialogue UI)

### Phase 4: Test Features (Week 5)
1. Word → Definition quiz
2. Definition → Word quiz
3. Fill-in-the-blank quiz (easy/hard)
4. Score tracking per test type

### Phase 5: Monetization (Week 6)
1. AdMob integration (banner ads)
2. StoreKit 2 (IAP setup)
3. Premium feature gates
4. Paywall UI

### Phase 6: Premium + Apple Intelligence (Week 7)
1. Home widget (WidgetKit)
2. Lock screen widget
3. Advanced statistics (Charts framework)
4. Custom themes
5. iCloud sync
6. Image quiz (ImageCreator API — iPhone 15 Pro+)
7. Real-time AI sentences (Foundation Models — iOS 26+)
8. Voice answer mode (Speech framework + Foundation Models)
   - User speaks the word's meaning aloud
   - Speech framework converts voice → text (on-device)
   - Foundation Models judges conceptual correctness vs actual definition
   - Correct → next card + familiarity +0.2
   - Incorrect → show correction + familiarity -0.2
   - Fallback: feature hidden on non-Apple Intelligence devices

### Phase 7: Polish & Launch (Week 8)
1. App icon & branding
2. Onboarding flow
3. App Store screenshots
4. App Store description
5. TestFlight beta
6. Submit to App Store

---

## Review Workflow

Every phase follows this cycle before moving to the next:

1. Claude Code implements the phase
2. Sahn takes simulator screenshots
3. **Design Review** — share screenshots with PM (Claude.ai) to check:
   - Layout, spacing, typography
   - Dark mode
   - Edge cases (long words, missing data)
4. **Product Review** — PM checks:
   - Feature completeness vs spec
   - Philosophy alignment (simple, offline-first, no over-engineering)
   - UX flow
5. **Fix sprint** — PM writes fix prompts → Claude Code implements
6. **Sign-off** → next phase begins

### For Claude Code
- At the end of each phase, add `🔍 Awaiting review` status to PROGRESS.md
- Do not start the next phase until PROGRESS.md shows `✅ Review passed`
- PM will update PROGRESS.md with review notes and sign-off

---

## Design Guidelines

### Color Scheme
- **Primary:** Inspired by classic index cards (cream/beige)
- **Accent:** Minimalist (blue for actions)
- **Dark Mode:** Fully supported

### Typography
- **System Font:** San Francisco (iOS default)
- **Card Text:** Clear, readable, 18-20pt
- **Pronunciation:** Monospace for IPA

### UI Principles
- **Minimalist:** No clutter
- **Focused:** One task per screen
- **Fast:** Immediate feedback
- **Accessible:** VoiceOver support, Dynamic Type

---

## Marketing Strategy

### App Store Optimization (ASO)
- **Keywords:** vocabulary, flashcards, SAT, GRE, TOEFL, learn words, no subscription
- **Screenshots:** 
  1. "Why pay subscription?" comparison
  2. Category selection
  3. Flashcard view
  4. Conversation example / Fill-in-the-blank test
  5. Progress tracking

### Launch Strategy
1. Product Hunt launch
2. Reddit (r/SAT, r/GRE, r/languagelearning)
3. Twitter/X promotion
4. Email to education influencers
5. App Store Today feature (pitch)

### Pricing Psychology
- Free tier: Full functionality (with ads)
- $2.99 premium: Impulse buy pricing
- Comparison: "Save $57/year vs Vocabulary.com"

---

## Key Decisions & Rationale

### Why SQLite instead of CloudKit database?
- **Offline-first:** Works without internet
- **Fast:** Local queries, no latency
- **Simple:** No backend complexity
- **CloudKit:** Only for version updates, not primary storage

### Why no daily habit features?
- **Philosophy:** Users study when they have goals, not daily
- **Reality:** Daily streaks create guilt, not learning
- **Differentiation:** Vocabulary.com failed with this approach

### Why $2.99 one-time instead of subscription?
- **User-friendly:** No recurring charges
- **Differentiation:** Main competitor charges $60/year
- **Sustainability:** Minimal server costs (CloudKit free tier)
- **Ethics:** Fair pricing for what's essentially a static database

### Why Free Dictionary API instead of premium APIs?
- **Cost:** $0 vs thousands
- **Quality:** Good enough for MVP
- **License:** Open source friendly (CC BY-SA)
- **Fallback:** Can supplement with WordNet, CMU Dictionary

### Why NGSL/NAWL/WordNet instead of Oxford 5000?
- **License:** Oxford 5000 prohibits commercial use
- **NGSL:** CC BY-SA 4.0 — fully open for commercial use
- **WordNet Core:** BSD license — fully open for commercial use
- **Coverage:** NGSL + NAWL + WordNet Core ≈ 8,000+ unique words

### Why pre-generate conversations and questions?
- **Offline-first:** No API dependency
- **Consistency:** Same quality for all users
- **Cost:** $0 vs per-request API costs
- **Apple Intelligence:** Used only for real-time enhancement on supported devices

### Why difficulty AND tags separately?
- **Orthogonal info:** difficulty = how hard, tags = which exam
- Not all hard words are GRE words; not all SAT words are easy
- Tags = high-frequency on that specific exam only

### Why AVSpeechSynthesizer for TTS?
- **Free:** $0 cost
- **Offline:** No internet needed
- **Quality:** Neural voices since iOS 16 — sufficient for vocabulary app
- **No Apple Intelligence required:** Works on all devices

### Why implicit tracking instead of "Know / Don't Know" buttons?
- **Frictionless:** No decision fatigue; swiping is the natural gesture anyway
- **Honest signal:** Hesitation and exploration (tapping examples) reveal more than a self-reported button press
- **Simpler UI:** No extra button row on each card; keeps the interface minimal
- **Familiar pattern:** Inspired by how physical flashcard timing works

### Why SwiftData for user data (not SQLite)?
- **Simplicity:** No manual SQL for CRUD; SwiftData integrates with SwiftUI
- **iCloud sync ready:** SwiftData + CloudKit sync is one flag away for premium
- **Separation of concerns:** Vocabulary DB is bundled read-only; user data is writable and local-only
- **Type safety:** @Model classes are type-checked at compile time

### Why GitHub Releases for DB updates (not CloudKit)?
- **Free:** No server or CloudKit storage costs for binary distribution
- **Simple versioning:** Release tag = DB version; no separate version.json needed
- **Transparent:** Public repo means users can inspect what changed
- **Reliable CDN:** GitHub's CDN handles distribution globally

---

## Common Tasks

### Add a new word category
1. Update word lists in `VocabularyData/word-lists/english/`
2. Run `update_db.py` to add words directly to `english_vocabulary.db`
3. Replace DB in Xcode project
4. Update `CategorySelectionView.swift`

### Update vocabulary data
1. Run `update_db.py` to modify `english_vocabulary.db` directly
2. Increment version in the DB `metadata` table
3. Copy updated `english_vocabulary.db` to `../2by3Words-db/`
4. Create a new GitHub Release in `scha00/2by3-words-db` with the new version tag

### Test TTS for a word
```swift
import AVFoundation

let synthesizer = AVSpeechSynthesizer()
let utterance = AVSpeechUtterance(string: "ephemeral")
utterance.voice = AVSpeechSynthesisVoice(language: "en-US")
utterance.rate = 0.5
synthesizer.speak(utterance)
```

### Query words by tag (high-frequency exam words)
```swift
let words = try db.prepare(
    wordsTable.filter(tags.like("%GRE%"))
                .order(difficulty.desc())
                .limit(20)
)
```

### Query words by difficulty range
```swift
let words = try db.prepare(
    wordsTable.filter(difficulty >= 7 && difficulty <= 10)
                .order(difficulty.asc())
)
```

### Check Apple Intelligence availability
```swift
import FoundationModels

switch SystemLanguageModel.default.availability {
case .available:
    // Show AI-enhanced features
case .unavailable:
    // Fall back to pre-generated content
}
```

---

## Resources

### APIs & Data Sources
- Free Dictionary API: https://dictionaryapi.dev/
- NGSL Project: https://www.newgeneralservicelist.com
- Princeton WordNet Core: https://wordnetcode.princeton.edu/standoff-files/core-wordnet.txt
- CMU Pronouncing Dictionary: https://github.com/cmusphinx/cmudict

### iOS Development
- SwiftUI Documentation: https://developer.apple.com/documentation/swiftui
- Foundation Models: https://developer.apple.com/documentation/FoundationModels
- ImagePlayground / ImageCreator: https://developer.apple.com/documentation/ImagePlayground/ImageCreator
- WidgetKit: https://developer.apple.com/documentation/widgetkit
- StoreKit 2: https://developer.apple.com/documentation/storekit
- AdMob iOS: https://developers.google.com/admob/ios/quick-start

### Design Resources
- SF Symbols: https://developer.apple.com/sf-symbols/
- Human Interface Guidelines: https://developer.apple.com/design/human-interface-guidelines/

---

## GitHub Repository Structure

This project uses **two repositories**:

### Private Repo — `scha00/2by3-words`
Source code. Not publicly accessible.

```
2by3-words/
├── .gitignore
├── README.md
├── CLAUDE.md              # This file — architecture & rules for both PM and Eng
├── PROGRESS.md            # Symlinked or copied from 2by3-words-db
├── LICENSE
├── twobythreewords/       # Xcode project
└── VocabularyData/        # Data pipeline scripts (no DB files here)
```

### Public Repo — `scha00/2by3-words-db`
Vocabulary database + project progress. Publicly readable.

```
2by3-words-db/
├── README.md
├── PROGRESS.md            # Project progress — readable by PM (Claude.ai)
├── en_words.db
├── en_examples.db
└── en_questions.db
```

Future languages follow the same pattern:
```
ko_words.db / ko_examples.db / ko_questions.db
es_words.db / es_examples.db / es_questions.db
```

GitHub Releases in this repo serve the DB files as downloadable assets.

### Why two repos?
- **Privacy:** iOS source code stays private
- **DB distribution:** App downloads `english_vocabulary.db` directly from public repo releases
- **Progress tracking:** `PROGRESS.md` is publicly readable so PM (Claude.ai) can fetch it by URL without copy-pasting
- **Simplicity:** No JSON intermediary — scripts write directly to SQLite

### Local clone layout
```
~/Projects/
├── 2by3Words/         # private — Swift code + scripts
└── 2by3Words-db/      # public — DB files + PROGRESS.md
```

### .gitignore (private repo — `2by3-words`)
```
# Xcode
*.xcuserstate
xcuserdata/
DerivedData/

# DB files live in public repo, not here
*.db

# Python
__pycache__/
*.pyc

# macOS
.DS_Store
```

### .gitignore (public repo — `2by3-words-db`)
```
.DS_Store
```

---

## Notes for Claude Code

### When working on this project:
1. **Always check CLAUDE.md first** for project context
2. **Follow the established structure** (don't create new patterns)
3. **Use SwiftUI** (not UIKit)
4. **Keep it simple** (anti-over-engineering philosophy)
5. **Test with real data** (use sample SQLite database)
6. **Consider offline-first** (don't assume network)
7. **Respect the philosophy** (goal-oriented, not daily habits)
8. **Apple Intelligence is additive** — always implement pre-generated fallback first
9. **Two storage layers:** SQLite (read-only vocabulary DB) + SwiftData (writable user data)
10. **No explicit know/don't know buttons** — familiarity is inferred from behavior

### Before making changes:
- Read relevant sections of CLAUDE.md
- Understand the "why" behind decisions
- Check if it aligns with project philosophy
- Consider user experience (simplicity first)

### When stuck:
- Review "Key Decisions & Rationale" section
- Check "Common Tasks" for examples
- Refer to data structure documentation
- Ask: "Does this add real value or complexity?"

### Two-repo workflow
- **This repo (`2by3-words`):** All Swift code and data scripts
- **Public repo (`2by3-words-db`):** DB files, PROGRESS.md, and CLAUDE.md
- Clone both repos side by side: `~/Projects/2by3Words/` and `~/Projects/2by3Words-db/`
- Never commit `.db` files to this repo — they belong in `2by3Words-db/`
- Always update `../2by3Words-db/PROGRESS.md` at the end of every session
- **CLAUDE.md sync:** Whenever CLAUDE.md is modified, manually copy it to `../2by3Words-db/CLAUDE.md` and push — both files must stay in sync

### PM communication protocol
- The PM (Claude.ai) reads progress by fetching: `https://raw.githubusercontent.com/scha00/2by3-words-db/main/PROGRESS.md`
- To flag something for the PM, add it to "Notes for PM" section in PROGRESS.md
- End every session with:
```
## Session summary
- ✅ Completed: [what you finished]
- 🔄 In progress: [what's partially done, or "None"]
- ❌ Blocked: [any blockers, or "None"]
- 📝 PROGRESS.md: updated
```

---

## Contact & Ownership

**Developer:** Sahn  
**Domain:** twobythreewords.com  
**Bundle ID:** com.sahn.twobythreewords  
**App Store:** 2by3 Words  

---

*Last Updated: 2026-03-22*
*Version: 1.6*

---

## UI Design Decisions

### Onboarding Flow (3 screens)
1. **Interactive Card Preview**
   - 실제 작동하는 단어 카드를 작게 보여줌 (북마크, 발음, 예시 탭 가능)
   - 위아래 스와이프로 단어 넘기기 직접 체험
   - 오른쪽 스와이프 → 다음 온보딩 화면
   - 하단에 "No streaks. No guilt. Study when you need to." 문구
   - Page indicator (dot)

2. **Level Selection**
   - 🌱 Elementary (Grades 1–3): cat, run, happy
   - 📗 Upper Elementary (Grades 4–6): vivid, scarce
   - 📘 Intermediate (High School) [기본값]: inevitable, keen
   - 🎓 Advanced (SAT/GRE level): ephemeral, astute
   - 각 레벨에 실제 단어 예시 표시
   - Skip → Intermediate 자동 적용

3. **직접 단어 카드 진입** (별도 화면 없이)

### Main Screen Layout
```
상단: [2by3 Words title]                    [iCloud sync icon]
중간: TTUIWordCard (위아래 스와이프, 탭 플립)
카드하단: TTUICardActionBar [Detail] [Dialogue] [Progress] [Bookmark]
하단: TTUITabBar [Deck] [Test] [Stats] [Settings]
```

### Card Interaction
- **위 스와이프**: 다음 단어
- **아래 스와이프**: 이전 단어
- **탭**: 앞면 ↔ 뒷면 플립 (3D rotation)
- **앞면**: 단어 + 발음기호(🔊) + 품사 + 대표뜻 (마스크 모드 지원)
- **뒷면**: Full 정의 + 예시 문장

### Mask Mode
- 앞면의 대표뜻을 blur 처리
- 탭하면 reveal
- 카드 상단에 eye.slash 토글로 전체 on/off

### Card Action Buttons (카드 하단 4개)
| 버튼 | 기능 |
|------|------|
| Detail | 사전형 풀 정의, 동의어/반의어 |
| Dialogue | 대화 예시 + AI TTS 오디오 재생 |
| Progress | 이 단어의 학습 성취도 (조회수, 테스트 정답률) |
| Bookmark | 북마크 토글 |

### Profile Icon (상단 우측)
- 탭 불가 (static display)
- CloudKit 동기화 상태 표시
  - `icloud` → 동기화 완료
  - `icloud.slash` → iCloud 꺼짐
  - `arrow.triangle.2.circlepath.icloud` → 동기화 중
- iCloud 없으면 로컬 저장으로 graceful fallback

### Data Sync Strategy
```swift
// 설정값: NSUbiquitousKeyValueStore (자동 iCloud 동기화)
// - userLevel, userGoals, isPremium, onboardingCompleted

// 학습 데이터: CloudKit CKRecord (프리미엄만)
// - 북마크, user_progress, 복습 스케줄

// 무료: 로컬 SQLite only
// 프리미엄: SQLite + CloudKit 동기화
```

### Tab Bar Structure
| 탭 | 내용 |
|----|------|
| Deck | 덱 스위처 + 단어 목록 + 북마크 + 덱 편집(프리미엄) |
| Test | contextual examples 기반 객관식/빈칸 테스트 |
| Stats | 학습 통계, 진도, 복습 필요 단어 |
| Settings | 레벨/목표 설정, 프리미엄, iCloud, 라이선스 |

### Multi-Deck (프리미엄)
- 기본: English 덱 1개
- 프리미엄: 여러 덱 생성/전환
- 상단 pill 버튼으로 덱 스위치: [English GRE ▾]
- 향후: Korean, Spanish 덱 추가 가능한 구조

### TTUI Framework
- **Prefix:** `TTUI`
- **위치:** `twobythreewords/twobythreewords/TTUI/`
- 모든 컴포넌트에 Xcode Preview 포함
- **Foundation 토큰:** `TTUIColor`, `TTUISpacing`, `TTUITypography`, `TTUIRadius`, `TTUISize`, `TTUIAnimation`
- **주요 컴포넌트:** `TTUIWordCard`, `TTUICardActionBar`, `TTUITabBar`
- **Colors:** Named color assets in `Assets.xcassets` — light + dark mode for all 14 colors
- **Card flip:** `TTUIFlipEffect` AnimatableModifier with `rotation3DEffect`
