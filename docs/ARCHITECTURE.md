# Architecture

This document covers the technical decisions, data models, and key design patterns for the Turn Zero application.

---

## Goals & Constraints

| Goal | Detail |
|---|---|
| **Mobile-first** | The app is used at a game table. Players look at it on a phone. Touch-friendly, large targets, minimal typing. |
| **Offline-capable** | A game shop may have poor Wi-Fi. Core functionality must work without a network connection. |
| **Two-player aware** | Both players make decisions; the app must clearly indicate whose turn it is at every point. |
| **Rules-enforcing** | The app should prevent illegal actions (e.g., a player taking 3 modifications) rather than being a passive checklist. |
| **Single source of truth** | Game state lives in one place; all UI is derived from it. |

---

## Technology Decisions

### Vite + React + TypeScript

- Lightning-fast dev server (HMR) — no server-side rendering needed for a game companion tool
- Pure client-side SPA is the right fit: all state is local, no data fetching from a server
- TypeScript enforces correct game state shapes throughout

### React Router v7

- Client-side routing between setup steps
- Familiar API, widely documented
- Nested routes map cleanly to the setup sub-steps

### Tailwind CSS

- Rapid mobile UI development
- Dark mode support out of the box (important for a dimly lit game store)
- No runtime CSS-in-JS overhead
- Built-in animation utilities (`animate-spin`, `transition-*`, `duration-*`) plus custom `@keyframes` in `tailwind.config.ts` cover all MVP animation needs — no additional animation library required

### React Context + `useReducer`

- Built into React — no additional package needed
- A single `GameSessionContext` wraps the app and provides state + dispatch to all route components
- All state mutations are named reducer actions, which maps cleanly onto the Turn Zero state machine (each step transition, each deck operation, each modification action is a discrete action type)
- `localStorage` persistence is handled by a `useEffect` that serializes state on every change and rehydrates on mount — ~5 lines, no middleware needed

### Motion (prev. Framer Motion)

- Step transition animations make it clear to players the flow is progressing
- Dice roll animation adds tactile feel to the Blue Player determination step

> **Note:** Motion is deferred post-MVP. Tailwind's built-in animation utilities and custom `@keyframes` cover all MVP animation needs. Motion can be revisited in Phase 9 (Polish) if exit animations are desired.

### Vitest + React Testing Library

- Game logic in `lib/game/` is pure functions — unit tested independently of React
- Component tests cover critical user interactions (modification turns, dice tiebreakers)

---

## Application Layers

```
┌─────────────────────────────────────────────────────┐
│  UI Layer  (React components / Tailwind)            │
│  • Step pages, dice roller, mission dashboard       │
│  • Reads from context, dispatches actions           │
├─────────────────────────────────────────────────────┤
│  State Layer  (React Context + useReducer)          │
│  • GameSessionContext — full Turn Zero state        │
│  • Persisted to localStorage via useEffect          │
├─────────────────────────────────────────────────────┤
│  Game Logic Layer  (pure TypeScript, no React)      │
│  • dice.ts      — black attack die simulation       │
│  • deck.ts      — Battle Deck operations            │
│  • setup-machine.ts — step transitions & guards     │
│  • terrain.ts   — terrain classification helpers    │
└─────────────────────────────────────────────────────┘
```

---

## Data Models

### Player

```typescript
interface Player {
  id: 'blue' | 'red';
  name: string;
}
```

### Terrain Piece

```typescript
type TerrainCoverage = 'light' | 'heavy' | 'area' | 'impassable';

interface TerrainPiece {
  id: string;
  name: string;
  coverage: TerrainCoverage;
  placed: boolean;
}
```

### Black Attack Die Result

```typescript
type DieFace = 'blank' | 'surge' | 'hit' | 'crit';

interface DiceRollResult {
  player: 'blue' | 'red';
  rolls: DieFace[];   // always length 4
  crits: number;
  hits: number;
  surges: number;
}
```

### Battle Deck

```typescript
type DeckType = 'objective' | 'secondary_objective' | 'advantage';

interface BattleCard {
  id: string;
  name: string;
  type: DeckType;
  setupInstructions?: string;
}

interface PlayerDeck {
  drawPile: BattleCard[];
  discardPile: BattleCard[];
  revealed: BattleCard | null;
}

interface PlayerBattleDecks {
  objective: PlayerDeck;
  secondaryObjective: PlayerDeck;
  advantage: PlayerDeck;
}
```

### Mission Dashboard

```typescript
interface MissionDashboard {
  objectiveCard: BattleCard | null;       // one card, revealed by one player
  secondaryObjectiveCard: BattleCard | null;
  blueAdvantageCard: BattleCard | null;
  redAdvantageCard: BattleCard | null;
  bluePlayerSide: 'blue' | 'red';         // can change during modification
}
```

### Mission Modification

```typescript
type ModificationAction =
  | 'replace_objective'
  | 'replace_secondary_objective'
  | 'replace_own_advantage'
  | 'replace_opponent_advantage'
  | 'claim_blue_player'
  | 'pass';

interface ModificationRecord {
  player: 'blue' | 'red';
  action: ModificationAction;
  turn: number;  // 1–4
}
```

### Setup Effects

```typescript
type SetupKeyword =
  | 'bounty'
  | 'cache'
  | 'covert_ops'
  | 'complete_the_mission'
  | 'field_commander'
  | 'hunted'
  | 'scouting_party'
  | 'transport';

interface SetupEffect {
  keyword: SetupKeyword;
  player: 'blue' | 'red';
  unitName: string;
  resolved: boolean;
}
```

### Prepared Position Unit

```typescript
interface PreparedPositionUnit {
  player: 'blue' | 'red';
  unitName: string;
  deployed: boolean;
}
```

### Game Session (root context shape)

```typescript
type SetupStep =
  | 'army_built'
  | 'battlefield_set'
  | 'terrain_declared'
  | 'terrain_placed'
  | 'blue_player'
  | 'mission_initial'
  | 'mission_modify'
  | 'mission_finalize'
  | 'setup_effects'
  | 'prepared_positions'
  | 'complete';

interface GameSession {
  id: string;
  createdAt: string;
  currentStep: SetupStep;

  players: {
    blue: Player;
    red: Player;
  };

  terrain: TerrainPiece[];

  bluePlayerRoll: DiceRollResult | null;
  redPlayerRoll: DiceRollResult | null;
  bluePlayerDetermined: boolean;
  rollHistory: DiceRollResult[][];   // each element is one round of rerolls

  blueDecks: PlayerBattleDecks;
  redDecks: PlayerBattleDecks;
  missionDashboard: MissionDashboard;
  modificationHistory: ModificationRecord[];
  modificationsRemaining: { blue: number; red: number };
  currentModifyingPlayer: 'blue' | 'red';

  setupEffects: SetupEffect[];

  preparedUnits: PreparedPositionUnit[];
  currentDeployingPlayer: 'blue' | 'red';
}
```

---

## State Machine

The `SetupStep` type defines all valid states. Transitions are guarded — a step cannot be advanced until its completion condition is met.

```typescript
// lib/game/setup-machine.ts
const STEP_ORDER: SetupStep[] = [
  'army_built',
  'battlefield_set',
  'terrain_declared',
  'terrain_placed',
  'blue_player',
  'mission_initial',
  'mission_modify',
  'mission_finalize',
  'setup_effects',
  'prepared_positions',
  'complete',
];

function canAdvance(session: GameSession): boolean {
  switch (session.currentStep) {
    case 'terrain_declared':
      return session.terrain.length > 0;
    case 'terrain_placed':
      return session.terrain.every(t => t.placed);
    case 'blue_player':
      return session.bluePlayerDetermined;
    case 'mission_initial':
      return (
        session.missionDashboard.objectiveCard !== null &&
        session.missionDashboard.secondaryObjectiveCard !== null &&
        session.missionDashboard.blueAdvantageCard !== null &&
        session.missionDashboard.redAdvantageCard !== null
      );
    case 'mission_modify':
      return (
        session.modificationsRemaining.blue === 0 &&
        session.modificationsRemaining.red === 0
      );
    case 'setup_effects':
      return session.setupEffects.every(e => e.resolved);
    case 'prepared_positions':
      return session.preparedUnits.every(u => u.deployed);
    default:
      return true;
  }
}
```

---

## Deck Operations

All deck mutation is handled through pure functions to keep the store actions thin.

```typescript
// lib/game/deck.ts

function revealTopCard(deck: PlayerDeck): { card: BattleCard; deck: PlayerDeck } | null {
  if (deck.drawPile.length === 0) {
    // Reshuffle discard pile (including the card just discarded, handled by caller)
    const reshuffled = shuffle([...deck.discardPile]);
    deck = { drawPile: reshuffled, discardPile: [], revealed: deck.revealed };
    if (deck.drawPile.length === 0) return null;
  }
  const [card, ...rest] = deck.drawPile;
  return { card, deck: { ...deck, drawPile: rest, revealed: card } };
}

function discardRevealed(deck: PlayerDeck): PlayerDeck {
  if (!deck.revealed) return deck;
  return {
    ...deck,
    discardPile: [...deck.discardPile, deck.revealed],
    revealed: null,
  };
}
```

---

## Routing Structure

```
/                         → Landing page (new game / resume game)
/setup                    → Turn Zero flow root (redirects to current step)
/setup/terrain            → Terrain declaration & placement
/setup/blue-player        → Dice roll
/setup/mission            → Mission dashboard + modification
/setup/effects            → Setup effects checklist
/setup/deployment         → Prepared positions
/setup/complete           → Summary & hand-off to game
```

All `/setup/*` routes read from and write to the same `GameSessionContext`. Navigation between sub-routes is controlled by the step machine (the URL does not drive state; state drives the URL). React Router's `<Navigate>` is used to redirect the user back to their current step if they try to deep-link ahead.

---

## Persistence Strategy

| Scenario | Strategy |
|---|---|
| Single device, two players | Both players share one device; state in `localStorage` via `useEffect` |
| Two devices, local network | Future: WebSocket room via a lightweight server |
| Two devices, remote play | Future: Supabase Realtime or Pusher |

For the initial release, single-device mode with `localStorage` persistence covers the primary use case (sitting across a table).

---

## Testing Strategy

| Test type | Coverage target | Location |
|---|---|---|
| Unit | All game logic functions (dice, deck, state machine guards) | `src/lib/game/__tests__/` |
| Component | Step pages — happy path + key error states | `src/components/setup/__tests__/` |
| Integration | Full Turn Zero flow from step 1 to complete | `src/__tests__/` |

Game logic is intentionally decoupled from React so it can be tested without any rendering overhead.

---

## Docker Architecture

This is a pure static SPA — Docker's role is dev environment consistency and portable production serving, not running application server logic.

### Production Image (multi-stage)

```
┌─────────────────────────────────────┐
│  Stage 1: builder  (node:22-alpine) │
│  • npm ci                           │
│  • npm run build  →  /app/dist      │
└─────────────────┬───────────────────┘
                  │ COPY dist/
┌─────────────────▼───────────────────┐
│  Stage 2: production (nginx:stable- │
│           alpine)                   │
│  • Serves /usr/share/nginx/html     │
│  • nginx.conf: SPA fallback +       │
│    immutable asset cache headers    │
│  • Exposes port 80                  │
└─────────────────────────────────────┘
```

The final image contains only Nginx and the compiled static assets — no Node, no source code. Typical image size: ~25 MB.

### Nginx SPA Routing

React Router v7 uses client-side routing. Without the `try_files` fallback, a hard refresh on any route other than `/` would 404. `nginx.conf` handles this:

```nginx
location / {
    try_files $uri $uri/ /index.html;
}
```

Static assets (JS, CSS, fonts, images) are served with `Cache-Control: public, immutable` and a 1-year expiry — safe because Vite content-hashes all asset filenames at build time.

### Docker Compose Services

| Service | Base image | Port | Profile | Purpose |
|---|---|---|---|---|
| `dev` | `node:22-alpine` | 5173 | *(default)* | Vite dev server with HMR; `src/` volume-mounted |
| `prod` | Built from `Dockerfile` | 8080→80 | `prod` | Local production image verification |

The `node_modules` volume prevents the host and container copies from conflicting when running `dev`.

### Dev Container

`.devcontainer/devcontainer.json` targets `node:22-alpine` directly (not Compose) so VS Code's Remote Containers extension can attach and install extensions inside the container. Port 5173 is forwarded automatically and `npm install` runs on container creation.

