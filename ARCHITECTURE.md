# FearBuster Code Architecture

## Entry & Root

**`src/main.tsx`** → Vite entry. Mounts React app into `<div id="root">`.

**`src/App.tsx`** → Root component. Manages view state machine:
- Renders Welcome, HowItWorks, current Stage, or Summary based on `view` state
- Wires up practice session hook, stage navigation, export panels
- Renders persistent UI: Pause & Ground, Safety & Privacy dialogs, top-right control links

**`src/types.ts`** → Complete TypeScript schema:
- `Practice` — the full session object (id, created/updated dates, current stage, completed stages, all responses)
- `PracticeResponses` — all user input across 8 stages
- `AppSettings` — autosave enabled, has-seen flags
- `View` — discriminated union of view states (welcome | howItWorks | stage | summary)

---

## Content & Configuration (Non-Code)

All UI text and mappings live here, separated from component logic so they can be edited without touching code:

**`src/content/stages.ts`**
- `STAGES` — array of 8 stage metadata (number, movement, title, progressLabel)
- Movement labels (Storm, Orientation, Passage, Return)
- Helper: `stageMeta(n)` to get metadata for stage N

**`src/content/copy.ts`**
- Welcome, HowItWorks, Stage 1–8, Grounding, Summary copy
- All UI prompts, helpers, buttons, placeholder text
- Organized by screen/stage so changes are easy to locate

**`src/content/beliefs.ts`**
- `BELIEF_OPTIONS` — 7 beliefs + their deterministic support statements
- `OTHER_BELIEF_ID` / `OTHER_BELIEF_STATEMENT` — fallback for "something else"
- No dynamic generation; all text reviewed and fixed

**`src/content/safety.ts`**
- Crisis resources (988, Crisis Text Line, emergency)
- Non-diagnostic, static, US-focused
- No keyword triggers

**`src/content/privacy.ts`**
- Autosave, data deletion, discreet exit copy
- Accurate language about localStorage scope and browser history

**`src/content/completion.ts`**
- `closingLine(practice)` — deterministic function
- Takes the completed practice, applies 4 rules (wrote own statement, named small step, built plan, default)
- Returns the single line shown on the summary screen

---

## Components

### Screens

**`src/components/Welcome.tsx`**
- Hero text, tagline, description
- "Begin" button, "How it works" link
- If recovery practice exists, inline recovery banner

**`src/components/HowItWorks.tsx`**
- Four movement cards (Storm, Orientation, Passage, Return) with short descriptions
- Privacy note, grounding availability note
- Back button, Begin button

**`src/components/Summary.tsx`**
- Closing affirmation (from `closingLine()`)
- Return plan snippet (what user will do right after)
- "Export this practice" button, "Start new", "Return to beginning" link
- Opens `ExportPanel` when export is clicked

### Stage Layout & Shell

**`src/components/StageShell.tsx`**
- Wraps every stage: progress bar, title, content, back/next buttons
- Handles stage navigation, continuation gating
- Used by Stages 1–8

**`src/components/ProgressIndicator.tsx`**
- Text label: "Stage 3 of 8: Meaning Beneath the Fear · Orientation"
- Thin fill-rule bar (8 segments, filled/unfilled based on current stage)
- No color-only or icon-only signaling
- Accessible via role="progressbar", aria-valuenow, aria-valuetext

### Individual Stages

**`src/components/stages/SingleFieldStage.tsx`** (Stages 1, 2, 3)
- Generic reusable layout: label + helper + single textarea
- Used by Fear Name, Body Sensation, Meaning Beneath Fear

**`src/components/stages/Stage4.tsx`**
- Three-column sorting exercise
- Columns: Realistic | Fear Adding | Unknown
- Responsive: full columns on desktop (md+), vertical stack on mobile
- Uses `Field` component (from `ui.tsx`)

**`src/components/stages/Stage5.tsx`**
- Radio button list of 7 beliefs + "something else"
- On selection, deterministically looks up and displays the support statement
- User can Accept, Edit, or Write Own
- Edit mode is an inline textarea; Write Own starts fresh
- Tracks `acceptedAsGiven` flag

**`src/components/stages/Stage6.tsx`**
- Three fields, always stacked: Values | Coping Resources | Small Step
- Each is a separate `Field` component with label, helper, textarea

**`src/components/stages/Stage7.tsx`**
- Return plan: three fields, stacked
- Immediate After | Who to Contact | What Helps Afterward
- Each is a `Field` component

**`src/components/stages/Stage8.tsx`**
- Optional closing reflection field
- Can be left blank; doesn't gate continuation

### Persistent UI

**`src/components/PauseGround.tsx`**
- Fixed pill button, bottom-right
- Radix Dialog (accessible, focus-trapped)
- Displays the 4-step grounding sequence
- "Return to where I was" button closes the dialog
- Always available (Stages 1–8)

**`src/components/SafetyNotice.tsx`**
- Radix Dialog modal
- US crisis resources (988, emergency, Crisis Text Line)
- Non-diagnostic notice, dismissible
- Opened via "Safety" link (top-right)

**`src/components/PrivacySettings.tsx`**
- Radix Dialog modal
- Autosave toggle (Radix Switch)
- Delete this practice, delete all practices (with confirmation dialogs)
- Discreet exit button
- Accurate storage & history language
- Opened via "Privacy" link (top-right)

**`src/components/SessionRecoveryBanner.tsx`**
- Shown on Welcome screen if an unfinished practice exists
- Displays date only (no sensitive content)
- Options: Resume, Review or Export, Start Fresh (with confirmation), Delete (with confirmation)

### Data Export

**`src/components/ExportPanel.tsx`**
- Radix Dialog
- Checkboxes to select which sections to export (Fear, Weight, Meaning, Sorting, Support, Values/Step, Return Plan, Closing)
- Buttons: "Download as text", "Download as PDF"
- PDF export shows loading state while jsPDF is imported dynamically

### UI Primitives

**`src/components/ui.tsx`**
- `PrimaryButton`, `SecondaryButton`, `TextLink` — styled button components
- `Field` — label + helper + textarea, used across all stages
- All built on Tailwind classes, no external component library except for Radix Dialog/Switch

---

## State Management & Persistence

**`src/hooks/usePractice.ts`** — Central hook managing all practice session state:
- Loads settings and practices from localStorage on mount
- Detects unfinished practice and offers recovery
- Methods:
  - `startNewPractice()` — creates a new Practice, saves if autosave on
  - `resumeActivePractice()` — loads the latest unfinished practice
  - `updateResponses(patch)` — updates current stage responses, triggers autosave
  - `goToStage(n)` — moves to stage N, marks previous as completed
  - `completePractice()` — marks practice as complete, force-saves to disk
  - `toggleAutosave(enabled)` — updates settings
  - `deletePracticeById(id)`, `deleteAllPractices()` — cleanup

**`src/lib/storage.ts`** — Low-level localStorage interface:
- `loadSettings()`, `saveSettings()`
- `loadPractices()`, `savePractices()`
- `getActivePractice()` — finds the most recent unfinished practice
- `upsertPractice()`, `deletePractice()` — immutable list operations
- `isStorageAvailable()` — checks if localStorage works (useful for private browsing)

---

## Export & PDF Generation

**`src/lib/exportContent.ts`** — Builds export sections from a completed practice:
- `buildExportSections(practice)` → array of `ExportSection` objects
- Each section has key, label, and array of lines (the user's responses formatted)
- Used by both text and PDF export

**`src/lib/textExport.ts`**
- `exportAsText(practice, selectedKeys)` → triggers a file download
- Formats sections as plain text, one section per newline group
- Creates blob, generates download link, auto-downloads

**`src/lib/pdfExport.ts`**
- `exportAsPdf(practice, selectedKeys)` → async, dynamically imports jsPDF
- Uses jsPDF to construct a PDF document
- Lays out sections with title, formatted text, page breaks
- Auto-downloads the PDF file

---

## Type Flow

```
App.tsx
  ├─ usePractice hook
  │   └─ returns: practice (Practice | null), updateResponses(), goToStage(), etc.
  │
  ├─ welcomeScreen
  │   └─ recoveryPractice (Practice | null)
  │
  ├─ currentStage (Stage 1–8)
  │   ├─ responses: practice.responses (PracticeResponses)
  │   ├─ onChange handlers call: updateResponses({ fieldKey: value })
  │   └─ canContinue checked against responses for gate logic
  │
  ├─ summaryScreen
  │   ├─ closingLine(practice) → string
  │   ├─ exportPanel references: buildExportSections(practice)
  │   └─ exportAs{Text,Pdf}(practice, selectedKeys)
  │
  └─ persistent UI (Pause&Ground, Safety, Privacy, Recovery Banner)
      └─ read/write via usePractice hook methods
```

---

## Data Flow (Happy Path)

1. **Welcome** → User clicks "Begin"
2. `usePractice.startNewPractice()` creates a new Practice, saves to localStorage
3. **Stage 1** → User types fear name
   - `onChange` → `updateResponses({ fearName: value })`
   - Hook updates practice, triggers autosave
4. **Stage 2–3** → Same pattern (bodySensation, meaningBeneathFear)
5. **Stage 4** → Three-field sorting
   - `onChange` → `updateResponses({ sorting: { realistic, exaggerated, unknown } })`
6. **Stage 5** → Belief selection + support statement
   - Radio selection → `updateResponses({ support: { beliefId, statementText, acceptedAsGiven } })`
7. **Stage 6** → Values, coping, small step
   - Three separate onChange calls, each updating a field
8. **Stage 7** → Return plan
   - `updateResponses({ returnPlan: { immediateAfter, whoToContact, selfCare } })`
9. **Stage 8** → Optional closing reflection
   - `updateResponses({ closingReflection: value })`
10. User clicks "Complete the practice"
    - `completePractice()` marks practice as complete, force-saves
    - View switches to Summary
11. **Summary** → User clicks "Export"
    - `ExportPanel` opens, user selects sections
    - Calls `exportAsText()` or `exportAsPdf()`
    - File downloads to device
12. User clicks "Return to beginning" or "Start new"
    - View resets to Welcome

---

## Recovery Flow

1. App loads, `usePractice` detects unfinished practice
2. Banner shown on Welcome screen (date only, no content)
3. Options:
   - **Resume** → `resumeActivePractice()`, navigate to that stage
   - **Review or Export** → Open `ExportPanel` on the recovered practice
   - **Start Fresh** → Confirmation dialog, then `discardActivePractice()` + `startNewPractice()`
   - **Delete** → Confirmation dialog, then `discardActivePractice()`

---

## Styling & Tailwind

**`tailwind.config.js`**
- Extends with custom colors (slate, ink, ember, sage, mist, brick)
- Extends fontFamily (serif: Newsreader, sans: Inter)
- Registers keyframes for fade-in animation

**`src/index.css`**
- Tailwind directives (base, components, utilities)
- Base layer: root/body styles, focus-visible outline
- `@media (prefers-reduced-motion)` disables animations globally

**Component styles**
- All inline via Tailwind classes (no separate .css files)
- No CSS-in-JS library; Tailwind is sufficient

---

## Testing & QA Checklist

See **FEARBUSTER_IMPLEMENTATION.md** for the full testing roadmap.

Quick checklist:
- [ ] Complete a full 8-stage practice, export as text and PDF
- [ ] Session recovery: close mid-stage, reload, resume
- [ ] Pause & Ground: preserves work, returns to same stage
- [ ] Autosave toggle: disable, verify loss on reload; re-enable, verify persistence
- [ ] Mobile: Stage 4 stacks vertically on phone
- [ ] Keyboard: Tab through all fields, buttons, dialogs
- [ ] Screen reader: NVDA/JAWS reads stage prompts, progress, buttons

---

## Deployment

See **README.md** for deployment options (GitHub Pages, Netlify, Docker, etc.).

Production build:
```bash
npm run build
# Output in dist/ — pure static files, ready to serve
```

No server, no database, no build step on the host.

---

## Questions/Next Steps

1. Any changes to copy or beliefs? Edit `/src/content/` files and rebuild.
2. Color tweaks? Edit `tailwind.config.js` colors section.
3. Accessibility audit? Run WCAG checker, screen-reader test.
4. Mobile layout issue? Check Stage4 responsive breakpoint in `stages/Stage4.tsx`.

---

**Last updated:** September 2026  
**Status:** Ready for review
