# Turn Zero

> A companion app for *Star Wars*: Legion that guides players through the pre-game **Setup** sequence — commonly called "Turn Zero" — before the first round begins.

---

## What is Turn Zero?

In *Star Wars*: Legion, both players must complete a structured series of steps before any combat begins. This is informally called **Turn Zero** or **Setup**. It involves terrain declaration, determining who controls the mission (the "blue player"), building a mission from Battle Decks, resolving any pre-game keyword effects, and deploying units with the *Prepared Position* keyword.

This process involves alternating decisions, dice rolls, deck reveals, and contested modifications — all of which benefit from a dedicated digital assistant to track state, enforce rules, and keep both players on the same page.

---

## Features

| Feature | Description |
|---|---|
| **Step Tracker** | Visual progress indicator through all 8 Setup phases |
| **Blue Player Determination** | Simulated 4-die black attack dice roll with tiebreaker logic |
| **Mission Builder** | Virtual Battle Deck system (Objective, Secondary Objective, Advantage) with reveal, discard, and replace flow |
| **Mission Modification Tracker** | Enforces the 2-modification-per-player limit with turn order |
| **Setup Effects Checklist** | Prompt for relevant keyword effects (Bounty, Cache, Covert Ops, etc.) |
| **Prepared Position Deployment** | Alternating deployment tracker starting with blue player |
| **Terrain Tracker** | Record agreed terrain pieces and their rule classifications |
| **Session Persistence** | Game state saved locally so a session can survive a phone sleep |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | [Next.js 15](https://nextjs.org/) (App Router) + TypeScript |
| Styling | [Tailwind CSS](https://tailwindcss.com/) |
| State Management | [Zustand](https://zustand-demo.pmnd.rs/) |
| Animations | [Framer Motion](https://www.framer.com/motion/) |
| Testing | [Vitest](https://vitest.dev/) + [React Testing Library](https://testing-library.com/) |
| Deployment | [Vercel](https://vercel.com/) |

---

## Getting Started

### Prerequisites

- Node.js ≥ 20
- npm ≥ 10 (or pnpm / yarn)

### Install & Run

```bash
npm install
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

### Build for Production

```bash
npm run build
npm start
```

---

## Project Structure

```
turn-zero/
├── app/                    # Next.js App Router pages & layouts
│   ├── (game)/             # Game session routes
│   │   ├── setup/          # Turn Zero step-by-step flow
│   │   └── session/[id]/   # Shared session view
│   └── page.tsx            # Landing / new game screen
├── components/             # Reusable UI components
│   ├── setup/              # Step-specific components
│   └── ui/                 # Generic design system primitives
├── lib/
│   ├── game/               # Core game logic (pure functions)
│   │   ├── dice.ts         # Black attack dice simulation
│   │   ├── deck.ts         # Battle deck shuffle / reveal logic
│   │   └── setup-machine.ts# Turn Zero state machine
│   └── store/              # Zustand stores
├── docs/                   # Supporting documentation
└── public/                 # Static assets (dice faces, token images)
```

---

## Documentation

| Document | Description |
|---|---|
| [Setup Rules Reference](docs/SETUP_RULES.md) | Full breakdown of the Turn Zero sequence and the game logic the app must implement |
| [Architecture](docs/ARCHITECTURE.md) | Technical decisions, data models, and state machine design |
| [Implementation Plan](docs/IMPLEMENTATION_PLAN.md) | Phased development roadmap with acceptance criteria |

---

## Reference

Rules sourced from the official *Star Wars*: Legion rules and [Legion Helper](https://legion.takras.net/setup/).

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) (coming soon). All game logic lives in `lib/game/` and must be covered by unit tests before merging.