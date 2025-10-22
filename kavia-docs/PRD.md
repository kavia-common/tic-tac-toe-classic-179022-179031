# Tic Tac Toe Frontend — Product Requirements Document (PRD)

## Overview
This document defines the product requirements for a single-container Vue 3 web application that delivers a classic Tic Tac Toe experience with Player vs Player (PvP) and Player vs Bot modes. The application follows the Ocean Professional style with a modern, minimalistic aesthetic, blue and amber accents, responsive layout, and smooth transitions. The UI centers around a responsive game card containing player controls, game mode and difficulty selectors, the board, status, and score display.

## Goals
- Provide an intuitive Tic Tac Toe experience with PvP and versus Bot modes.
- Offer three bot difficulty levels: Easy, Medium, and Hard.
- Maintain a polished visual style aligned with Ocean Professional.
- Ensure responsive design for mobile, tablet, and desktop.
- Track scores across games within the session using persistent storage (localStorage).
- Enable quick game restart and full score reset.

## Non-Goals
- Online multiplayer or networked games.
- User authentication and profiles.
- External data persistence beyond browser localStorage.
- Accessibility beyond core keyboard operability and ARIA basics already implemented.
- Advanced analytics or telemetry.

## User Stories
- As a player, I can select PvP or play against a Bot so that I can choose my preferred mode.
- As a player in Bot mode, I can choose Easy, Medium, or Hard to match my desired challenge.
- As a player, I can see whose turn it is and the current status (turn, win, draw).
- As a player, I can make moves on a 3x3 grid and receive immediate feedback.
- As a player, I can start a new game quickly without losing the current series scores.
- As a player, I can reset scores to start a new series from zero.
- As a player, I can view scores for X, O, and draws that persist across page reloads in the same browser.

## Requirements

### Functional Requirements
- Game Modes:
  - PvP mode: Two human players alternate moves on the same device.
  - PvBot mode: Player X vs Bot O (bot moves are automated after the user’s turn).
  - Bot Difficulty:
    - Easy: Random valid moves.
    - Medium: Win/block heuristics with preference (center > corners > sides).
    - Hard: Minimax with alpha–beta pruning for optimal play.
- Game Rules:
  - Standard 3x3 Tic Tac Toe rules, win on three in a row (rows, columns, diagonals).
  - Draw is detected when the board is full and no winner exists.
  - Win/draw detection must highlight winning line and update status text.
- Controls:
  - Mode selector: PvP or Bot.
  - Difficulty selector: Enabled only in Bot mode (Easy/Medium/Hard).
  - New Game: resets the board and alternates starting player for fairness.
  - Reset Scores: resets all scores (X, O, Draws) to 0 and clears board.
- Score Tracking:
  - Scores for X wins, O wins, and draws are displayed.
  - Scores persist in localStorage across page reloads in the same browser.
- Status Display:
  - Shows current turn, or “Winner: X/O!”, or “Draw!” based on state.
- Accessibility:
  - The board is implemented with semantic roles (role="grid" and role="gridcell").
  - Buttons have aria-labels indicating cell index and state.
  - Controls have accessible labels.
- Routing:
  - App supports a simple router with a Home view displaying the game and a basic About route.

### Non-Functional Requirements
- Performance:
  - Bot actions should be performant and feel instantaneous; a small delay (around 250ms) is acceptable for UX feedback.
- Reliability:
  - Deterministic bot behavior per difficulty strategy, no crashes on repeated fast clicks.
- Maintainability:
  - Code is modular with single-file components; bot strategies are encapsulated functions.
- Security:
  - No external communications; localStorage only.
- Compatibility:
  - Latest stable Chrome, Firefox, Safari, and Edge; responsive across common mobile/desktop sizes.
- Localization:
  - English only.

## Game Modes and Bot Difficulties
- PvP: Both players take turns as X and O; move validation prevents overwriting a cell.
- PvBot:
  - Easy: Chooses a random available cell.
  - Medium: 
    - If bot can win in one move, it plays that move.
    - Else, if the player can win next move, it blocks.
    - Else, prefers center, then corners, then sides.
  - Hard:
    - Full-depth minimax with alpha–beta pruning for optimal play.
    - Prefers center for the first move if available.

## UI and Layout (Ocean Professional)
- Layout:
  - A centered responsive card contains:
    - Top controls row: Mode and Difficulty selectors on the left; Reset Scores and New Game on the right.
    - Status text beneath controls.
    - A 3x3 grid board with clear separation and hover/disabled states.
    - A scoreboard row: X, Draws, O with numeric totals.
- Visual Design:
  - Colors: primary #2563EB (blue), secondary #F59E0B (amber), error #EF4444, background #f9fafb, surface #ffffff, text #111827.
  - Subtle gradients in backgrounds, rounded corners, and soft shadows.
  - Smooth focus and hover transitions to reinforce a modern, polished look.
- Theming:
  - Light theme implemented per Ocean Professional palette. Dark theme is noted as a potential future extension (see Future Extensions).
- Responsive Behavior:
  - Controls stack on narrow screens; board scales using aspect-ratio and clamp-based font sizing.
  - Touch-friendly targets and spacing maintained at mobile widths.

## Score Tracking
- The series scoreboard increments on each win or draw.
- New Game keeps scores but resets the board, alternating the first player.
- Reset Scores clears both the scoreboard and the board, starting fresh.
- Scores persist via localStorage key ttt_scores_v1.

## Restart/Reset
- New Game:
  - Clears the board, sets status to in-progress, and alternates the starting player.
  - If the game is Bot mode and bot starts, the bot takes the first turn automatically.
- Reset Scores:
  - Sets scores (X, O, Draws) to 0 and resets the board to initial state with X to start.

## Error Handling
- Inputs are constrained to valid board indices and disabled buttons prevent illegal interactions.
- LocalStorage parsing is guarded; non-numeric values are ignored gracefully.
- Bot move attempts are skipped when game is not in progress or no valid moves exist.

## Acceptance Criteria
- A player can select PvP or Bot mode and observe the difficulty selector enabling/disabling accordingly.
- The board prevents moves on occupied cells and when the game is over.
- The status text accurately reflects turn, win, or draw.
- Wins and draws update the scoreboard; scores are persisted across page reloads.
- New Game alternates the starting player; Reset Scores clears all scores to zero.
- Bot plays according to the selected difficulty and respects turn.
- The page renders correctly on mobile and desktop with polished Ocean Professional styling.

## Dependencies
- Vue 3, Vite, TypeScript, Pinia (installed), Vue Router, Vitest for unit tests.

## Out of Scope
- Multiplayer over network, ranked systems, sound/music, animations beyond subtle transitions, or bot personalities beyond the three defined strategies.

## Future Extensions
- Dark theme toggle and full theme variables to support OS-level prefers-color-scheme.
- Timers and move history.
- Multiple board sizes or game variants.
- Internationalization (i18n).
- Improved accessibility with full keyboard navigation across the grid and focus management for announcements.

Sources:
- src/components/TicTacToe.vue
- src/App.vue
- src/main.ts
- src/router/index.ts
- src/assets/main.css
- index.html
