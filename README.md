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
| Build Tool | [Vite](https://vitejs.dev/) + [React](https://react.dev/) + TypeScript |
| Routing | [React Router v7](https://reactrouter.com/) |
| Styling | [Tailwind CSS](https://tailwindcss.com/) (includes animations via utility classes + custom keyframes) |
| State Management | React Context + `useReducer` (built-in) — persisted to `localStorage` |
| Testing | [Vitest](https://vitest.dev/) + [React Testing Library](https://testing-library.com/) |
| Containerization | [Docker](https://www.docker.com/) — multi-stage build (Node 22 → Nginx) |
| Deployment | [Vercel](https://vercel.com/) / any static host / Docker image |

---

## Getting Started

### Prerequisites

- Node.js ≥ 22 (Active LTS)
- npm ≥ 10 (or pnpm / yarn)

### Install & Run

```bash
npm install
npm run dev
```

Open [http://localhost:5173](http://localhost:5173) in your browser.

### Build for Production

```bash
npm run build
npm run preview
```

---

## VS Code Setup

This repo ships a `.vscode/` folder with launch configurations and extension recommendations ready to go.

### Recommended Extensions

When you open the repo, VS Code will prompt you to install the recommended extensions. Accept the prompt, or install them manually via the Extensions panel (`⇧⌘X`) → *Show Recommended Extensions*:

| Extension | Purpose |
|---|---|
| **ESLint** (`dbaeumer.vscode-eslint`) | Inline lint errors |
| **Prettier** (`esbenp.prettier-vscode`) | Auto-format on save |
| **Tailwind CSS IntelliSense** (`bradlc.vscode-tailwindcss`) | Class autocomplete & hover docs |
| **Vitest** (`vitest.explorer`) | Run & debug tests from the Test Explorer sidebar |
| **Edge DevTools** (`ms-edgedevtools.vscode-edge-devtools`) | In-editor browser DevTools |
| **PostCSS Language Support** (`csstools.postcss`) | Syntax highlighting for PostCSS / Tailwind directives |

### Launch Configurations

Open the **Run and Debug** panel (`⇧⌘D`) and choose a configuration from the dropdown:

| Configuration | What it does |
|---|---|
| **Launch Chrome (Dev Server)** | Starts `npm run dev` then opens Chrome with the debugger attached. Set breakpoints directly in `.tsx` source files. |
| **Run Vitest (all tests)** | Runs the full test suite in the integrated terminal with verbose output. |
| **Debug Vitest (current file)** | Runs only the test file currently open in the editor — useful for focused debugging. |

> **Tip:** The Vitest extension (`vitest.explorer`) also adds inline ▶ run/debug buttons next to each `it()`/`test()` block in the editor gutter, which is the fastest way to run a single test case.

---

## Docker

The repo ships three Docker artifacts:

| File | Purpose |
|---|---|
| `Dockerfile` | Multi-stage production image (Node 22 build → Nginx static serve) |
| `nginx.conf` | Nginx config with SPA fallback routing and static asset caching |
| `docker-compose.yml` | `dev` service (hot reload) and `prod` service (production image preview) |
| `.devcontainer/devcontainer.json` | VS Code Dev Container — full Node 22 environment, no local install needed |

### Dev Container (Recommended for onboarding)

Open the repo in VS Code, then when prompted **"Reopen in Container"** — or run **Remote-Containers: Reopen in Container** from the command palette. VS Code will build the container, install dependencies, and forward port `5173` automatically.

### Docker Compose — Dev (hot reload)

```bash
docker compose up dev
```

The `src/` directory is volume-mounted so Vite HMR works exactly as it does locally. Open [http://localhost:5173](http://localhost:5173).

### Docker Compose — Production Preview

```bash
docker compose --profile prod up prod
```

Builds the full production image and serves it via Nginx at [http://localhost:8080](http://localhost:8080). Use this to verify the production build before deploying.

### Production Image (standalone)

```bash
docker build -t turn-zero .
docker run -p 8080:80 turn-zero
```

---

## Project Structure

```
turn-zero/
├── src/
│   ├── routes/             # React Router v7 route components
│   │   ├── index.tsx       # Landing / new game screen
│   │   └── setup/          # Turn Zero step pages
│   │       ├── terrain.tsx
│   │       ├── blue-player.tsx
│   │       ├── mission.tsx
│   │       ├── effects.tsx
│   │       ├── deployment.tsx
│   │       └── complete.tsx
│   ├── components/         # Reusable UI components
│   │   ├── setup/          # Step-specific components
│   │   └── ui/             # Generic design system primitives
│   ├── lib/
│   │   ├── game/           # Core game logic (pure functions)
│   │   │   ├── dice.ts     # Black attack dice simulation
│   │   │   ├── deck.ts     # Battle deck shuffle / reveal logic
│   │   │   └── setup-machine.ts  # Turn Zero state machine
│   │   └── store/          # Zustand stores
│   ├── router.tsx          # React Router configuration
│   └── main.tsx            # App entry point
├── docs/                   # Supporting documentation
├── public/                 # Static assets (dice faces, token images)
├── index.html              # Vite HTML entry point
└── vite.config.ts
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