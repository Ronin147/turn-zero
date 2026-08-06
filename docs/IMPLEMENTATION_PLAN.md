# Implementation Plan

This document lays out the phased development roadmap for Turn Zero. Each phase has a clear goal, a list of deliverables, and acceptance criteria. Phases build on each other — complete each phase before starting the next.

---

## Phase 0 — Project Scaffolding

**Goal:** A running Vite + React app with the chosen toolchain wired up and CI passing.

### Tasks

- [ ] Bootstrap Vite + React + TypeScript project (`npm create vite@latest -- --template react-ts`)
- [ ] Add React Router v7 (`react-router-dom`)
- [ ] Configure Tailwind CSS (dark mode via `class` strategy; add custom `@keyframes` for dice roll animation)
- [ ] Configure Vitest + React Testing Library
- [ ] Add ESLint + Prettier with agreed rules
- [ ] Add `Dockerfile` (multi-stage Node 22 build → Nginx production image)
- [ ] Add `nginx.conf` (SPA fallback routing + immutable asset cache headers)
- [ ] Add `docker-compose.yml` (`dev` service with HMR volume mount; `prod` service under `prod` profile)
- [ ] Add `.devcontainer/devcontainer.json` for VS Code Dev Container support
- [ ] Set up GitHub Actions: lint → type-check → test → Docker build on every push
- [ ] Write a single smoke test that renders the root page

### Acceptance Criteria

- `npm run dev` starts the app with no errors, but we want to use the launch settings from VSCode
- `npm run build` produces a clean production build
- `npm test` runs and the smoke test passes
- `docker compose up dev` serves the app at `localhost:5173` with hot reload
- `docker compose --profile prod up prod` serves the production build at `localhost:8080`
- `docker build -t turn-zero .` produces an image under 50 MB
- Opening the repo in a VS Code Dev Container installs dependencies and forwards port 5173 automatically
- A GitHub Actions pipeline passes on the main branch
---

## Phase 1 — Landing Page & Session Management

**Goal:** Players can start a new game session and name themselves, and the session persists across page refreshes.

### Tasks

- [ ] Design the `GameSession` TypeScript interface (see `ARCHITECTURE.md`)
- [ ] Implement `GameSessionContext` with `useReducer` and `localStorage` persistence via `useEffect`
- [ ] Build `createSession(bluePlayerName, redPlayerName): GameSession` factory
- [ ] Landing page UI: player name inputs, "Start Game" button
- [ ] Resume banner: if a session exists in storage, offer to resume or start fresh
- [ ] Route to `/setup` after session creation
- [ ] Unit test: `createSession` returns correct initial state

### Acceptance Criteria

- Players can enter names and start a session
- Refreshing the browser resumes the session at the last step
- Starting fresh clears the old session
- All new-session unit tests pass

---

## Phase 2 — Turn Zero Step Tracker

**Goal:** The app displays the 8-step Setup sequence with a progress indicator. Players can advance through confirmation-only steps.

### Tasks

- [ ] Implement `SetupStep` state machine (`lib/game/setup-machine.ts`)
- [ ] `canAdvance(session)` guard function with tests for each step
- [ ] `advanceStep(session): GameSession` pure function
- [ ] Step progress bar / stepper component (mobile-friendly, shows step name and number)
- [ ] Step wrapper layout: title, description, content area, "Continue" button
- [ ] Pages for Steps 1 & 2 (army built + battlefield set — confirmation only)
- [ ] Unit tests: `canAdvance` returns correct boolean for each step given mock state

### Acceptance Criteria

- Players see a visual stepper showing current and completed steps
- Steps 1 and 2 advance on button confirmation
- "Continue" button is disabled when `canAdvance` returns false
- Tapping a completed step shows a read-only summary (non-destructive back navigation)

---

## Phase 3 — Terrain Declaration & Placement

**Goal:** Players can record agreed terrain pieces with classifications, then mark each as placed.

### Tasks

- [ ] `TerrainPiece` data model and terrain store actions
- [ ] Terrain declaration screen: add terrain form (name + coverage type selector)
- [ ] Terrain list display with edit/delete
- [ ] Terrain placement screen: list with "Mark as placed" toggle per piece
- [ ] `canAdvance` guard: step 3 requires ≥ 1 terrain piece; step 4 requires all placed
- [ ] Unit tests: terrain add, remove, place operations

### Acceptance Criteria

- Players can add N terrain pieces with a name and coverage type
- Placement screen clearly shows placed vs. unplaced
- Cannot advance past terrain placement until all pieces are marked placed
- Empty terrain list blocks advancement past declaration step with a visible explanation

---

## Phase 4 — Blue Player Determination

**Goal:** The app simulates rolling 4 black attack dice for each player, applies tiebreaker rules, and announces the blue player.

### Tasks

- [ ] `DieFace` type and `rollBlackDie(): DieFace` function with correct face distribution (2 blank, 2 hit, 1 surge, 1 crit)
- [ ] `rollFourDice(): DiceRollResult` function
- [ ] `determineBluePlayer(blue: DiceRollResult, red: DiceRollResult): 'blue' | 'red' | 'reroll'` with full tiebreaker logic
- [ ] Dice roller UI: animated dice faces, roll button per player (or "Roll for Both")
- [ ] Result display: show both results, highlight winning result type, announce blue player
- [ ] Reroll state: if fully tied, display reason and reroll button; store each round of rolls in history
- [ ] Unit tests: `determineBluePlayer` for all priority tiebreaker cases and full-tie case

### Acceptance Criteria

- Dice animation plays on roll
- Tiebreaker logic correctly identifies blue player for all scenarios
- Full-tie triggers a reroll prompt with explanation
- Roll history is visible (players can see previous rounds)
- After blue player is determined, "Continue" unlocks

---

## Phase 5 — Mission Building

**Goal:** The full mission build flow — initial reveal, 4-round modification, and final setup — is tracked correctly.

### Tasks

#### 5a — Battle Deck Simulation

- [ ] `BattleCard` and `PlayerDeck` models
- [ ] `shuffle<T>(arr: T[]): T[]` Fisher-Yates implementation with tests
- [ ] `revealTopCard(deck: PlayerDeck)` with automatic reshuffle-on-empty
- [ ] `discardRevealed(deck: PlayerDeck)` pure function
- [ ] Seed-data: a minimal set of representative card names for each deck type (real card names not required, just placeholders like "Objective A", "Advantage B")

#### 5b — Initial Reveal UI

- [ ] Mission dashboard component showing 4 card slots
- [ ] Blue player flow: choose Objective or Secondary Objective to reveal first
- [ ] Red player flow: automatically prompted to reveal the other type
- [ ] Both players reveal their Advantage cards
- [ ] Animate cards into their slots

#### 5c — Mission Modification

- [ ] `ModificationRecord` model and store actions
- [ ] `modificationsRemaining` tracker (starts at `{ blue: 2, red: 2 }`)
- [ ] Current player indicator (alternates starting with blue)
- [ ] Modification action menu: 6 options as described in `SETUP_RULES.md`
- [ ] "Claim Blue Player" action transfers the blue player token
- [ ] Empty deck reshuffle prompt and handling
- [ ] Lock modifications after 4 total; advance to finalize step
- [ ] Unit tests: modification turn order, claim blue player token transfer, empty deck handling

#### 5d — Mission Finalize

- [ ] Blue player selects long edge (visual battlefield diagram, tap to choose side)
- [ ] Display each card's setup instructions in order (Objective → Secondary Objective → Advantage cards)
- [ ] Mark each instruction as acknowledged before advancing

### Acceptance Criteria

- Mission dashboard accurately reflects the current state at all times
- Exactly 4 modifications are allowed (2 per player), in correct alternating order
- Claiming blue player correctly swaps the token and the subsequent turn order
- Empty deck triggers a reshuffle notification before the reveal
- Finalize step guides players through each card's instructions in the correct sequence

---

## Phase 6 — Setup Effects

**Goal:** Players declare which keyword effects apply to their army and resolve them in the correct order.

### Tasks

- [ ] `SetupEffect` model and store actions
- [ ] Setup effects checklist UI: per-player section, each keyword with a checkbox and short rules summary
- [ ] "Add unit" flow: player enters unit name + selects keyword
- [ ] Resolution prompt: for each unresolved effect (blue player first), show the rule and a "Resolved" button
- [ ] `canAdvance` guard: all effects resolved
- [ ] Unit tests: effect ordering (blue before red), resolution state transitions

### Acceptance Criteria

- Both players can declare which units have setup keywords
- Blue player's effects are presented first
- Each effect displays a plain-English rules summary so players don't need the rulebook
- Cannot advance until all effects are marked resolved

---

## Phase 7 — Prepared Position Deployment

**Goal:** Alternating deployment for units with the *Prepared Position* keyword is tracked to completion.

### Tasks

- [ ] `PreparedPositionUnit` model and store actions
- [ ] Both players declare their Prepared Position units (name entry)
- [ ] Deployment tracker: shows current player, remaining units for each player, alternating prompt
- [ ] "Deploy unit" action per turn
- [ ] Handle edge cases: one player has more units than the other (remaining units still deploy)
- [ ] Completion state: all units deployed, display "Setup Complete" summary

### Acceptance Criteria

- Deployment alternates correctly starting with the blue player
- When one player runs out of units, the other continues deploying theirs
- Tapping "Deploy" removes the unit from the pending list
- After all units are deployed, the "Setup Complete" view is shown

---

## Phase 8 — Setup Complete Summary

**Goal:** A final screen summarizes the completed setup so both players have a reference before Round 1.

### Tasks

- [ ] Summary screen: final mission cards (names + instructions), terrain list, blue player, deployment order
- [ ] "Start Round 1" button — clears the session from storage
- [ ] Option to copy a plain-text game summary to clipboard
- [ ] "New Game" button — returns to the landing page

### Acceptance Criteria

- All key setup decisions are visible on one scrollable screen
- "Start Round 1" clears session state; back navigation returns to the landing page (not back into setup)
- Clipboard copy works on mobile (navigator.clipboard API with fallback)

---

## Phase 9 — Polish & PWA

**Goal:** The app feels native, is installable on a phone, and works fully offline.

### Tasks

- [ ] Add `vite-plugin-pwa` for Service Worker and offline caching
- [ ] Web app manifest with icon set (legion-themed icon)
- [ ] Dark mode default (game store lighting)
- [ ] Accessibility audit: all interactive elements have correct ARIA labels, touch targets ≥ 44px
- [ ] Animate step transitions (Tailwind `transition-opacity`/`animate-in`; evaluate adding Motion for exit animations if needed)
- [ ] Haptic feedback on dice roll (navigator.vibrate where supported)
- [ ] Performance audit: Lighthouse score ≥ 90 on mobile

### Acceptance Criteria

- "Add to Home Screen" prompt appears on mobile Chrome/Safari
- App loads and functions fully with network disabled (airplane mode test)
- No accessibility errors in axe-core scan
- Lighthouse mobile score ≥ 90

---

## Future Phases (Backlog)

| Feature | Notes |
|---|---|
| **Real-time two-device sync** | WebSocket room or Supabase Realtime; both players use their own phone |
| **QR code session sharing** | Generate a QR code to join a session from a second device |
| **Army list import** | Import an army list (e.g., from Legion HQ or Tabletop Admiral) to auto-populate keyword effects and Prepared Position units |
| **Card database** | Searchable database of real Objective, Secondary Objective, and Advantage card names |
| **Game log** | Exportable record of all Turn Zero decisions for post-game reference |
| **Tournament mode** | Timer per phase, stricter validation for competitive play |

---

## Milestone Summary

| Milestone | Phases | Key Deliverable |
|---|---|---|
| **M1: Foundation** | 0–1 | Running app, CI/CD, session persistence |
| **M2: Core Flow** | 2–4 | Steps 1–5 playable end-to-end |
| **M3: Mission** | 5 | Full mission build + modification engine |
| **M4: Complete** | 6–8 | All 8 steps, setup complete summary |
| **M5: Ship** | 9 | PWA, offline, polished |
