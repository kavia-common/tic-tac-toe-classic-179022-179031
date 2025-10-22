# Tic Tac Toe Frontend — Architecture

## Overview
This document describes the architecture for the single-container Vue 3 web application implementing a Tic Tac Toe game with Ocean Professional styling. It covers component structure, state, bot AI strategy, routing, data flow, error handling, accessibility, testing approach, and future-proofing considerations.

## High-Level Structure
- Framework: Vue 3 + Vite + TypeScript
- State: Local reactive state within the TicTacToe component; Pinia is available and can be adopted for global needs.
- Routing: Vue Router with Home (game) and About routes.
- Styling: Ocean Professional palette via CSS variables and component-scoped styles, with responsive design patterns.

### Component Tree
- App.vue
  - Header (brand/title)
  - Content
    - TicTacToe (core game)
  - Footer

Additionally, HomeView uses TicTacToe for route-based rendering:
- router/index.ts routes `/` to HomeView.vue which renders TicTacToe.
- App.vue also renders TicTacToe directly; projects can consolidate to one approach if needed. Current app renders the game in both App.vue and HomeView.vue routes; the active route used at runtime is via router mounting. Recommended future consolidation: use only HomeView for the game and let App host <router-view/>.

## Component Responsibilities

### App.vue
- Provides application shell with header/footer and Ocean Professional background.
- Hosts the core game component in main content.

### TicTacToe.vue
- Manages all game logic and UI:
  - Board state (array of nine cells).
  - Current player, mode (pvp/bot), difficulty (easy/medium/hard).
  - Game status (in-progress, won, draw), winner, and winning line.
  - Scoreboard with localStorage persistence.
  - Controls for mode/difficulty selection, New Game, and Reset Scores.
  - Accessibility roles and aria-labels on board cells.

### HomeView.vue
- Route-level wrapper rendering TicTacToe for the home route.

## State Management Approach
- Local component state in TicTacToe.vue using Vue’s Composition API (ref, computed, watch, onMounted).
- Persistence:
  - Scores are persisted via localStorage key ttt_scores_v1.
- Rationale:
  - The application’s state is localized to the game, with no cross-component sharing needs. Pinia is initialized at app level and available for future expansions. A simple store is present as example (stores/counter.ts) but is not used by the game.

Potential future migration:
- If additional views or cross-page data are added (e.g., settings, history), move score/state handling to a dedicated Pinia store with typed actions.

## AI/Bot Difficulty Strategy
- Easy:
  - Randomly selects from available moves.
- Medium:
  - Attempts immediate win for O.
  - If not possible, blocks X’s immediate win.
  - Otherwise prefers center, then corners, then sides.
- Hard:
  - Uses minimax search with alpha–beta pruning to compute optimal moves.
  - Full-depth search is feasible for 3x3 Tic Tac Toe.
  - First move preference: center if available.

Key functions in TicTacToe.vue:
- availableMoves(b): returns open cell indices.
- evaluateWinner(b): returns winner and winning line if present.
- minimax(b, depth, isMax, alpha, beta): computes best score/move recursively.
- botMoveEasy/Medium/Hard(): strategy-specific move selection.
- botMove(): dispatches to the selected difficulty.

## Routing
- routes:
  - `/` -> HomeView.vue -> <TicTacToe/>.
  - `/about` -> AboutView.vue (lazy-loaded).
- History mode: createWebHistory with BASE_URL from Vite env.
- Recommendation:
  - Prefer App.vue as layout wrapper with <router-view/> to host views; keep the game mounted only in HomeView for a single source of rendering.

## Data Flow

### Sequence (PvBot, typical turn)
1. User clicks an empty cell:
   - makeMove(index) validates and records X move.
2. Post-move evaluation:
   - evaluateWinner -> if win: status=won, winner=X, increment X score.
   - Else if board full: status=draw, increment draws.
   - Else: currentPlayer toggles to O.
3. Bot turn:
   - botMove is invoked with a brief UX delay (~250ms).
   - Strategy computes move and calls makeMove for O.
4. Round completion:
   - Status reflects win/draw/in-progress; scoreboard updates accordingly.
   - New Game resets the board (alternating starter) while preserving scores; Reset Scores clears all.

### Persistence
- onMounted: load existing scores from localStorage.
- watch(scores, { deep: true }): persist scores on change.

## Error Handling
- Guard clauses:
  - Ignore clicks when game is not in progress or cell is occupied.
  - Skip bot moves when game finished or no moves available.
- Storage safety:
  - localStorage parsing in try/catch; validates numeric fields before applying.

## Accessibility
- Board container has role="grid" with aria-label.
- Cells are buttons with role="gridcell" and aria-label indicating position and state.
- Control labels and disabled states are used for clarity.
- Future improvements:
  - Provide full keyboard navigation between cells.
  - Announce status updates via ARIA live regions.

## Styling and Theming
- Ocean Professional variables in CSS:
  - --primary: #2563EB, --secondary: #F59E0B, --error: #EF4444, --background: #f9fafb, --surface: #ffffff, --text: #111827.
- Component card: rounded corners, subtle outer/inset shadows, gradient accents.
- Responsive:
  - Board uses aspect-ratio and clamp font sizing.
  - Controls stack on small screens via grid.

### Dark Mode (Planned)
- Not implemented yet. Future approach:
  - Define a dark theme :root[data-theme="dark"] variable set and toggle via a header button or OS preference (prefers-color-scheme).

## Testing Approach
- Unit Testing:
  - Vitest + Vue Test Utils configured.
  - Current sample test exists for HelloWorld component.
- Recommended tests for TicTacToe.vue:
  - Move validation: cannot click occupied cells.
  - Win detection: three-in-a-row recognition and highlighting.
  - Draw detection: full board with no winner.
  - Score updates: increments for X, O, and draws; persistence via localStorage (mocked).
  - New Game: board reset, alternating starter behavior.
  - Reset Scores: scores cleared and board reset.
  - Bot behaviors: Easy randomness (seed/mocking), Medium block/win heuristics, Hard optimality in standard scenarios.

## Future Extensions
- Theme toggle (light/dark), accessible high-contrast mode.
- Game history and replay step-through.
- Multiple board sizes or advanced variants.
- Settings view to configure starting player and bot speed.
- Pinia-based centralized state for multi-view features.
- E2E tests (Cypress/Playwright) for flows across routes and UI.

## Known Deltas vs. Ideal Architecture
- Rendering location:
  - The game is currently mounted directly in App.vue and also referenced via HomeView in the router. For a cleaner architecture:
    - App.vue should host <router-view/>, and TicTacToe should be rendered only in HomeView (or App should rely solely on routed content).
- Theming:
  - Only light theme is implemented; dark theme design variables are not yet defined.

Sources:
- src/components/TicTacToe.vue
- src/App.vue
- src/main.ts
- src/router/index.ts
- src/assets/main.css
- index.html
