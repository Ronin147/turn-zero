# Turn Zero: Setup Rules Reference

This document is the authoritative game-rules reference for everything the Turn Zero app must implement. It is derived from the official *Star Wars*: Legion rules and the [Legion Helper](https://legion.takras.net/setup/) resource.

---

## Overview — The 8 Setup Steps

| # | Step | App Involvement |
|---|---|---|
| 1 | Build an Army, a Command Hand, and a Battle Deck | Reference only (out of scope) |
| 2 | Establish the Battlefield and Gather Components | Prompt / checklist |
| 3 | Declare Terrain | Terrain tracker |
| 4 | Place Terrain | Prompt / checklist |
| 5 | Determine Blue Player | Dice roller + tiebreaker logic |
| 6 | Build a Mission | Full deck + modification engine |
| 7 | Resolve Setup Effects | Keyword checklist |
| 8 | Deploy in Prepared Positions | Alternating deployment tracker |

---

## Step 1 — Build an Army, a Command Hand, and a Battle Deck

Each player privately assembles:

- **Army** — units totaling up to the agreed point limit
- **Command Hand** — command cards chosen according to the force-building rules
- **Battle Deck** — three sub-decks, one of each type:
  - **Objective Deck**
  - **Secondary Objective Deck**
  - **Advantage Deck**

> **App role:** This step happens before the app is opened. The app may display a reminder checklist but does not manage army building.

---

## Step 2 — Establish the Battlefield and Gather Components

Players set up the physical play area:

- Standard battlefield is **3′ × 6′**
- Players gather tokens, dice, order tokens, wound tokens, and the **Mission Dashboard**

> **App role:** Display a checklist so nothing is forgotten before proceeding.

---

## Step 3 — Declare Terrain

Players negotiate and agree on all terrain pieces and their rules classifications:

| Classification | Cover provided |
|---|---|
| Light terrain | Light cover |
| Heavy terrain | Heavy cover |
| Area terrain | See terrain rules |
| Impassable | Blocks movement |

Players must agree on a classification for every piece **before** the game begins.

> **App role:** Allow players to name and classify terrain pieces. Store the agreed list for reference throughout the game.

---

## Step 4 — Place Terrain

Players **cooperatively** place all declared terrain onto the battlefield. There is no turn order for this step — both players work together.

> **App role:** Display the agreed terrain list as a placement checklist. Players mark each piece as placed.

---

## Step 5 — Determine Blue Player

### Procedure

1. Both players each roll **4 black attack dice** simultaneously.
2. Count results in this priority order to find the **blue player**:

| Priority | Result | Symbol |
|---|---|---|
| 1st | Critical hit | ✦ (crit) |
| 2nd | Hit | ● (hit) |
| 3rd | Surge | ↑ (surge) |

3. The player with the **most** results at the highest unfied priority wins blue player.
4. If all three result types are tied, **reroll all dice** and repeat.

### Possible black attack die face distribution

A standard black attack die has:

| Face | Count |
|---|---|
| Blank | 2 |
| Surge | 1 |
| Hit | 2 |
| Critical hit | 1 |

> **App role:** Provide a virtual dice roller for both players (or accept manual entry). Display results, apply tiebreaker logic automatically, and announce the blue player. Handle full reroll if all results are tied.

---

## Step 6 — Build a Mission

This is the most complex step of Turn Zero.

### 6a — Initial Mission Reveal

1. Place the **Mission Dashboard** with the mission side face up.
2. Each player separates their Battle Deck into its three sub-decks and **shuffles each** independently.
3. Place a **blue player token** on the blue player's side of the Mission Dashboard.
4. The **blue player** chooses to reveal the top card of **either** their Objective deck **or** Secondary Objective deck and places it in the corresponding slot on the Mission Dashboard.
5. The **red player** reveals the top card of whichever type the blue player did **not** reveal, and places it in the corresponding slot.
6. **Both players** each reveal the top card of their own Advantage deck and place them in the Advantage slots.

> After step 6, the Mission Dashboard shows:
> - 1 Objective card
> - 1 Secondary Objective card
> - 2 Advantage cards (one per player)

### 6b — Mission Modification

Starting with the **blue player**, players alternate performing modifications. **Each player modifies exactly twice** (4 total modifications), then the mission is locked.

#### Available Modification Actions (choose one per turn)

| # | Action | Notes |
|---|---|---|
| 1 | Replace the **Objective** card | Reveal top of your Objective deck, discard current, place new |
| 2 | Replace the **Secondary Objective** card | Reveal top of your Secondary Objective deck, discard current, place new |
| 3 | Replace **your own Advantage** card | Reveal top of your Advantage deck, discard current, place new |
| 4 | Replace **opponent's Advantage** card | Opponent reveals top of their Advantage deck; discard current, place new |
| 5 | **Claim blue player** | Move the blue player token to your side. You are now the blue player. |
| 6 | **Pass** | No effect. Still counts as one modification. |

#### Empty Deck Rule

If a player must reveal from a deck that is empty, they first **reshuffle all previously discarded cards** of that type (including the card just discarded) to form a new deck, then reveal the top card.

#### Modification Turn Order

```
Blue Player → Red Player → Blue Player → Red Player
(2 modifications each, 4 total)
```

> **App role:** Track whose turn it is to modify, count remaining modifications per player, manage each deck's draw pile and discard pile (separate per player per deck type), handle empty deck reshuffles, allow blue player token transfers, and lock the mission after 4 total modifications.

### 6c — Finalize Mission Setup

After mission modification is complete, set up the mission in this order:

1. The **blue player** chooses one of the **long edges** of the battlefield as their side. The opposite long edge is the red player's side.
   - Each player's **allied territory** is the half of the battlefield closest to their edge.
   - **Enemy territory** is the opponent's half.
2. Follow any **setup instructions on the Objective Card**.
3. Follow any **setup instructions on the Secondary Objective Card**.
4. Starting with the **blue player**, each player follows any **setup instructions on their Advantage Card**.

> **App role:** Display the finalized mission cards in order. Walk players through setup instructions one card at a time.

---

## Step 7 — Resolve Setup Effects

Starting with the **blue player**, players resolve all abilities and keyword effects that trigger during Setup.

### Keywords with Setup Effects

| Keyword | Summary |
|---|---|
| **Bounty** | At the start of the game, secretly assign a bounty target from the opponent's army |
| **Cache X** | At the start of the game, gain X additional tokens of the specified type |
| **Covert Ops** | At the start of the game, this unit may choose to not be a commander for victory conditions |
| **Complete the Mission** | Condition for end-of-game scoring tied to unit survival |
| **Field Commander** | Unit may issue orders as if it were a commander; designate at setup |
| **Hunted** | This unit has a bounty placed on it by the opponent |
| **Scouting Party X** | At the start of the game, allow X friendly units to place a free move |
| **Transport** | Allows a unit to begin the game embarked in a vehicle |

> **App role:** Ask both players if any units in their army have these keywords. For each confirmed keyword, display a prompt explaining the resolution and allow the player to mark it as resolved. Turn order is blue player first, then red player.

---

## Step 8 — Deploy in Prepared Positions

Units with the **Prepared Position** keyword may deploy before the normal deployment phase.

### Procedure

1. Starting with the **blue player**, players **alternate** placing one unit with Prepared Position at a time.
2. Units are placed within their player's **deployment zone** as defined by the Objective and Advantage cards.
3. Continue alternating until all Prepared Position units have been placed.

> **App role:** Track which units each player has flagged as having Prepared Position. Display an alternating deployment prompt, confirm each placement, and indicate when all prepared units have been placed.

---

## Objective Token Reference

Objective tokens are placed during mission setup and scored during the game. The app should display their placement rules but does not need to score them.

### Types

| Type | Description |
|---|---|
| **Asset Token** | A valuable object that can be carried by units |
| **Point-of-Interest (POI)** | Represents critical locations; treated as area terrain providing heavy cover for LOS/cover purposes. Cylinder silhouette: token-width diameter, Range ½ tall. Does not block LOS for other purposes. |

### POI Rules
- Miniatures, Advantage tokens, and other Objective tokens **cannot overlap** POIs.
- A POI **may** be represented by a miniature on a 2-inch base; it still counts as a token, not a miniature.

---

## State Machine Summary

The Turn Zero flow is a linear **state machine** with one branching point (tiebreaker reroll) and one sub-loop (mission modification):

```
START
  │
  ▼
[1] ARMY_BUILT          ──── (confirmation) ────►
[2] BATTLEFIELD_SET     ──── (confirmation) ────►
[3] TERRAIN_DECLARED    ──── (name + classify all terrain) ────►
[4] TERRAIN_PLACED      ──── (mark each piece placed) ────►
[5] BLUE_PLAYER         ──── (roll dice → tiebreaker if needed) ────►
[6a] MISSION_INITIAL    ──── (blue picks obj/sec-obj; red picks other; both reveal advantage) ────►
[6b] MISSION_MODIFY     ──── (4 modifications, alternating) ────────────────┐
                                                              (empty deck?)  │
                                                              └── reshuffle ─┘
[6c] MISSION_FINALIZE   ──── (blue picks side; follow card instructions) ────►
[7] SETUP_EFFECTS       ──── (keyword prompts, blue first) ────►
[8] PREPARED_POSITIONS  ──── (alternating deployment, blue first) ────►
DONE (game begins)
```
