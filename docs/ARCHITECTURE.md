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

### Next.js 15 (App Router) + TypeScript

- Server components for the landing/lobby page
- Client components for the stateful Turn Zero flow (no server round-trips during gameplay)
- TypeScript enforces correct game state shapes throughout

### Tailwind CSS

- Rapid mobile UI development
- Dark mode support out of the box (important for a dimly lit game store)
- No runtime CSS-in-JS overhead

### Zustand

- Lightweight, no-boilerplate state management
- Easy to persist to `localStorage` with `persist` middleware
- One store per game session

### Framer Motion

- Step transition animations make it clear to players the flow is progressing
- Dice roll animation adds tactile feel to the Blue Player determination step

### Vitest + React Testing Library

- Game logic in `lib/game/` is pure functions — unit tested independently of React
- Component tests cover critical user interactions (modification turns, dice tiebreakers)

---

## Application Layers

```
┌─────────────────────────────────────────────────────┐
│  UI Layer  (React components / Tailwind)            │
│  • Step pages, dice roller, mission dashboard       │
│  • Reads from store, dispatches actions             │
├─────────────────────────────────────────────────────┤
│  Store Layer  (Zustand)                             │
│  • gameSessionStore  — full Turn Zero state         │
│  • Persisted to localStorage                        │
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

### Game Session (root store shape)

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
/setup                    → Turn Zero flow (client component, full state)
/setup/terrain            → Terrain declaration & placement
/setup/blue-player        → Dice roll
/setup/mission            → Mission dashboard + modification
/setup/effects            → Setup effects checklist
/setup/deployment         → Prepared positions
/setup/complete           → Summary & hand-off to game
```

All `/setup/*` routes read from and write to the same Zustand store. Navigation between sub-routes is controlled by the step machine (the URL does not drive state; state drives the URL).

---

## Persistence Strategy

| Scenario | Strategy |
|---|---|
| Single device, two players | Both players share one device; state in `localStorage` |
| Two devices, local network | Future: WebSocket room via a lightweight server |
| Two devices, remote play | Future: Supabase Realtime or Pusher |

For the initial release, single-device mode with `localStorage` persistence covers the primary use case (sitting across a table).

---

## Testing Strategy

| Test type | Coverage target | Location |
|---|---|---|
| Unit | All game logic functions (dice, deck, state machine guards) | `lib/game/__tests__/` |
| Component | Step pages — happy path + key error states | `components/setup/__tests__/` |
| Integration | Full Turn Zero flow from step 1 to complete | `app/__tests__/` |

Game logic is intentionally decoupled from React so it can be tested without any rendering overhead.
