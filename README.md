# Fantasy Cricket League — PyQt5 + SQLite

A desktop **Fantasy Cricket League** application with a **PyQt5 GUI** and a **SQLite** backend.  
Users can **sign up / sign in**, **create fantasy teams** under a points budget, **save & reopen teams**, and **evaluate team scores** for a selected match using stored match statistics.

This repo also includes the original **problem statement PDF(s)** used for the project.

---

## Key features

- **User authentication** (Sign Up / Sign In) backed by SQLite
- **Create Team** with constraints:
  - pick exactly **11 players**
  - enforce **category limits** (BAT / BWL / AR / WK)
  - enforce **points budget** (starts at 100; points used/available tracked)
- **Save Team** to database and reuse later
- **Open Team** to view previously saved selections
- **Evaluate Results**:
  - select a match + team
  - compute **fantasy points** per player and total team score from match stats

---

## What I did (in order)

1. Designed and wired up a **multi-window PyQt5 GUI** (auth → home → create/open/evaluate flows).
2. Implemented a **SQLite database schema** to store users, player stats, match performance, and saved teams.
3. Built the **team creation workflow** with:
   - player filtering by category
   - validation for 11-player selection
   - point-budget tracking (points used / points remaining)
4. Implemented **team persistence** (saving and reopening teams from SQLite).
5. Implemented **match-based evaluation** and a scoring engine to compute per-player and total team points.
6. Packaged everything into runnable Python files generated from `.ui` screens and connected signals/slots.

---

---

## How the app works (high-level)

### 1) Authentication
- Users sign up / sign in.
- Credentials are stored in the `extras` table.
- The app tracks the **currently logged-in user** using `current_username`.

### 2) Team creation
- Pick players from available lists by category (BAT/BWL/AR/WK).
- Each player has a **point value** (`PValue`) from `stats`.
- The app updates:
  - **Points available** (starts at 100)
  - **Points used**
  - **Category counts**
- Save is enabled only when the team is valid (11 players and within budget).

### 3) Saving teams
- Teams are stored in a per-user table named like:
  - `teams_of_<username>`
- A new team is created by adding a **new column** to that table with the team name, then inserting selected players into that column.

### 4) Evaluation
- Pick a team + match number.
- For each player in the team, the app fetches match stats from `matches` and calculates fantasy points.
- Shows per-player points and the **total team score**.

---

## Database schema (important tables)

### `extras`
Stores users and session-like fields:
- `username`, `password`
- `current_team`
- `points_available` (default 100)
- `points_used` (default 0)
- `current_username` (tracks who is logged in)

### `stats`
Player master data:
- `Player`, `Category` (BAT/BWL/AR/WK), `PValue`
- plus historic batting fields like matches/runs/50s/100s (used mainly for player/value metadata)

### `matches`
Per-match performance used in evaluation:
- `Match_Number`, `Player`
- batting: `Scored`, `Faced`, `Fours`, `Sixes`
- bowling/fielding: `Bowled`, `Maidens`, `Given`, `Wickets`, `Catches`, `Stumpings`, `RunOuts`

### `teams_of_<username>`
Per-user saved teams table.  
Implementation detail: each team is stored as a **column** with rows containing player names (and the last row storing points used).

---

## Scoring rules (implemented in `eval2.py`)

Points are computed per player using match stats. Examples from the implemented logic:

- Runs: `0.5 × Scored`
- Fours: `+1` each
- Sixes: `+2` each
- Maidens: `+5` each
- Wickets: `+10` each
- Fielding (catches + stumpings + runouts): `+3` each

Bonus points:
- Half-century: `+5` (50–99 runs)
- Century: `+10` (100+ runs)
- Strike rate bonuses (if balls faced != 0)
- 3-wicket haul and 5-wicket haul bonuses
- Economy rate bonuses (if overs bowled != 0)

Total team score = sum of all selected players’ points for the chosen match.

---

## Setup and run

### Requirements
- Python 3.x
- PyQt5

Install:
 pip install PyQt5

Start from the login screen:
 python sign2in5.py
