# Captain SONAR — Digital Adaptation

A **Unity-based multiplayer digital adaptation** of the board game *Captain Sonar*, built with **Photon Networking** for real-time online play. Two teams command rival submarines, each with specialized crew roles, competing to locate and destroy the opponent before being sunk themselves.

---

## 📋 Table of Contents

- [About the Game](#about-the-game)
- [Game Principles](#game-principles)
- [Architecture Overview](#architecture-overview)
- [Project Structure](#project-structure)
- [Roles & Classes](#roles--classes)
- [Core Systems](#core-systems)
- [Getting Started](#getting-started)
- [Tech Stack](#tech-stack)
- [License](#license)

---

## 🎮 About the Game

**Captain Sonar** is a team-based strategy game for 2–8 players divided into two teams. Each team operates a submarine and must work together to find and destroy the enemy submarine. The game can be played in **turn-based** or **real-time** mode.

This project is a **digital reimplementation** of the 2nd edition rules, developed as an academic/UX design project using Unity.

---

## 🧭 Game Principles

### Teams & Submarines

- Two teams (Red and Blue), each controlling a submarine.
- Each submarine has **4 health points**. When health reaches 0, the submarine is destroyed and the team loses.
- Submarines navigate a shared grid-based map divided into **sectors**.
- Submarines start submerged and can **surface** to reset certain states — but surfacing leaves the crew unable to act for a number of turns.

### Movement

- The Captain orders movement in one of four cardinal directions: **North, South, East, West**.
- A submarine **cannot** revisit a tile it has already passed through (its trail), nor move onto islands.
- Movement is tracked on each team's board.

### Game Modes

- **Turn-based**: Teams take turns. Each team cycles through its roles in order (Captain → First Mate → Engineer → Radio Detector).
- **Real-time** *(planned)*: Both teams play simultaneously with a shared timer.

### Winning

The first team to sink the opposing submarine wins!

---

## 🏗️ Architecture Overview

The project follows a **component-based architecture** typical of Unity, with a **singleton GameManager** orchestrating the game loop. Networking is handled via **Photon Unity Networking (PUN)** for multiplayer synchronization.

### Key Design Patterns

- **Singleton**: `GameManager` uses a singleton pattern to ensure a single authoritative game state.
- **Abstract Base Class (Role)**: All crew roles inherit from an abstract `Role` class, enforcing a consistent interface (`PerformRoleAction`, `FinishTurn`, `ToggleUI`, etc.).
- **Component Pattern**: Players, Roles, and Submarines are Unity `MonoBehaviour` components composed on GameObjects.
- **Coroutine-based Turn System**: The turn-by-turn flow uses Unity coroutines (`IEnumerator`) to wait for player input asynchronously.
- **Manager Classes**: Dedicated managers handle specific subsystems (e.g., `GridManager`, `CaptainGridManager`, `EngineerBoardManager`, `LogManager`).

---

## 📁 Project Structure

```
Assets/
├── Prefabs/
│   ├── CaptainBoard        # Captain's game board UI
│   ├── EngineerBoard        # Engineer's game board UI
│   ├── FirstMateBoard       # First Mate's game board UI
│   ├── RadioDetectorBoard   # Radio Detector's game board UI
│   ├── Tile                 # Grid tile prefab
│   ├── Timer                # Countdown timer prefab
│   ├── dialButton           # Directional dial buttons
│   └── Log Content          # Log display prefab
├── Scripts/
│   ├── Managers/
│   │   ├── GameManager.cs       # Core game loop & singleton
│   │   ├── GridManager.cs       # Grid generation & management
│   │   ├── CaptainGridManager.cs
│   │   ├── EngineerBoardManager.cs
│   │   └── LogManager.cs
│   ├── Roles/
│   │   ├── Role.cs              # Abstract base class for roles
│   │   ├── Captain.cs           # Captain role implementation
│   │   ├── FirstMate.cs         # First Mate role implementation
│   │   ├── Engineer.cs          # Engineer role implementation
│   │   └── RadioDetector.cs     # Radio Detector role implementation
│   ├── Elements/
│   │   ├── Submarine.cs         # Submarine entity
│   │   ├── Player.cs            # Player entity
│   │   ├── Board.cs             # Board display logic
│   │   ├── Tile.cs              # Tile logic
│   │   ├── Position.cs          # 2D coordinate struct
│   │   ├── Systems.cs           # Weapons & abilities system
│   │   ├── Timer.cs             # Turn countdown timer
│   │   └── DragObject.cs        # Drag interaction handler
│   └── ...
└── Scenes/
├── TestGame
├── TestCaptain
└── ...
```

---

## 🎭 Roles & Classes

Each team has **four crew roles**, each with a dedicated board and specific responsibilities:

### Captain (`Captain.cs`)
- **Responsibility**: Directs the submarine's trajectory and is the central link between all crew members.
- **Actions**:
  - Choose the submarine's initial position on the map.
  - Order movement in a cardinal direction (N/S/E/W).
  - Activate weapon systems (Mine, Torpedo, Sonar, Drone).
  - Order the submarine to surface.
- Inherits from `Role`.

### First Mate (`FirstMate.cs`)
- **Responsibility**: Manages the submarine's system gauges (weapons and special abilities).
- **Actions**:
  - Fill gauges for systems after each movement.
  - Report system readiness to the Captain.
- Inherits from `Role`.

### Engineer (`Engineer.cs`)
- **Responsibility**: Manages the submarine's mechanical health and breakdowns.
- **Actions**:
  - Mark breakdowns on the engineering board after each movement.
  - Repair systems when conditions allow.
- Inherits from `Role`.

### Radio Detector (`RadioDetector.cs`)
- **Responsibility**: Tracks the enemy submarine's movements.
- **Actions**:
  - Listen to the enemy Captain's announced directions.
  - Plot the enemy's probable path on a transparent overlay grid.
- Uses a doubled-size grid (via `GridManager`) to account for any possible starting position.
- Inherits from `Role`.

### Abstract Role (`Role.cs`)
All roles inherit from the abstract `Role` class which provides:
- Turn management (`ToggleTurn`, `FinishTurn`).
- A coroutine-based action system (`PerformRoleAction`).
- Abstract methods for UI toggling and role description.

---

## ⚙️ Core Systems

### GameManager
- **Singleton** managing the entire game lifecycle.
- Handles submarine instantiation, player assignment, camera distribution, turn cycling, and win conditions.
- Supports turn-based flow: Captain → First Mate → Engineer → Radio Detector, then switches teams.

### Submarine (`Submarine.cs`)
- Represents a team's submarine on the grid.
- Tracks health (max 4 HP), position, trail (visited tiles), and submersion state.
- Provides `Move()`, `SetPosition()`, `TakeDamage()`, and `MakeSurface()` methods.

### Systems (`Systems.cs`)
Manages all **weapons and special abilities**, each with a color-coded gauge:

| System     | Color  | Gauge Size | Effect |
|------------|--------|------------|--------|
| **Mine**     | 🔴 Red    | 2 | Drop a mine at a nearby position (range ≤ 4). Can be detonated later. |
| **Torpedo**  | 🔴 Red    | 3 | Fire a torpedo at a position (range ≤ 4). Deals 2 damage on direct hit, 1 on near miss. |
| **Sonar**    | 🟢 Green  | 2 | Ask the enemy team for coordinates — one must be true, one false. |
| **Drone**    | 🟢 Green  | 3 | Ask if the enemy is in a specific sector — they must answer truthfully. |
| **Silence**  | 🟡 Yellow | 5 | Move 0–4 tiles in one direction without announcing it. |

Gauges are filled by the First Mate after each movement. A system can only be activated when its gauge is full **and** there is no breakdown (failure) on that color.

### Player (`Player.cs`)
- Represents an individual player.
- Can be assigned one or more roles and a submarine.
- Manages camera switching between role boards.

### Position (`Position.cs`)
Simple 2D coordinate class (`x`, `y`) used for submarine location, weapon targeting, and map interactions.

### Map & Grid
- The game map is a 2D integer matrix.
- Islands and obstacles are encoded in the grid.
- `GridManager` generates tile-based UI grids for each role's board.

---

## 🚀 Getting Started

### Prerequisites

- **Unity** version `2021.3.32f1` (LTS — highly recommended for stability)
- **GitHub Desktop** (or Git CLI)
- A **GitHub account**
- **Photon Unity Networking (PUN)** — included in the project

### Setup

1. **Fork** this repository (do not clone directly):
   - Click the **Fork** button on GitHub to create your own copy.
   - This lets you work independently and add your own contributors.

2. **Clone** your forked repo:
   - Open **GitHub Desktop** → File → Clone Repository.
   - Select your fork and choose a local folder.
   - ⚠️ Make sure you have enough disk space — Unity projects can be large!

3. **Open in Unity**:
   - Open **Unity Hub** → Click **Add Project** → Navigate to the cloned folder.
   - ⚠️ Ensure you are using Unity version **2021.3.32f1**.

4. **Run the game**:
   - Open the `TestGame` scene.
   - Press Play to start a local test session.

### Git Workflow

1. **Fetch** — Check if teammates have pushed changes.
2. **Pull** — Download any new changes (resolve merge conflicts if needed).
3. **Commit** — Stage your changes locally with a descriptive message.
4. **Push** — Upload your commits to the remote repository.

---

## 🛠️ Tech Stack

| Technology | Purpose |
|---|---|
| **Unity 2021.3.32f1** | Game engine |
| **C#** | Scripting language |
| **Photon Unity Networking (PUN)** | Multiplayer networking |
| **TextMeshPro (TMP)** | UI text rendering |
| **GitHub** | Version control & collaboration |

---

## ⚠️ Known Incomplete Features

Several systems are partially implemented and marked with `// À COMPLÉTER` in the code:

- **Silence** activation logic
- **Sonar** coordinate exchange flow
- **Drone** sector verification
- **Engineer** failure/breakdown mechanics (`NoFailure` method)
- **Captain** initial position selection via grid UI
- **Real-time mode** (currently only turn-based is functional)
- **Photon synchronization** for full online multiplayer

---

## 📄 License

This project is an academic/educational project. Please check with the project maintainers for licensing details before redistribution.
