<script lang="ts" setup>
// Ocean Professional themed Tic Tac Toe with PvP and Bot modes.
// Features: move validation, win/draw detection, restart, scoreboard with localStorage,
// AI difficulties: Easy (random), Medium (win/block heuristics), Hard (minimax with pruning).

import { ref, computed, watch, onMounted, nextTick } from 'vue'

type Cell = 'X' | 'O' | null
type Mode = 'pvp' | 'bot'
type Difficulty = 'easy' | 'medium' | 'hard'

const WIN_LINES: number[][] = [
  [0, 1, 2],
  [3, 4, 5],
  [6, 7, 8],
  [0, 3, 6],
  [1, 4, 7],
  [2, 5, 8],
  [0, 4, 8],
  [2, 4, 6]
]

// state
const board = ref<Cell[]>(Array(9).fill(null))
const currentPlayer = ref<'X' | 'O'>('X')
const mode = ref<Mode>('pvp')
const difficulty = ref<Difficulty>('easy')
const status = ref<'in-progress' | 'won' | 'draw'>('in-progress')
const winner = ref<'X' | 'O' | null>(null)
const winLine = ref<number[] | null>(null)

const scores = ref({ X: 0, O: 0, draws: 0 })

// load persisted scores
onMounted(() => {
  const raw = localStorage.getItem('ttt_scores_v1')
  if (raw) {
    try {
      const parsed = JSON.parse(raw)
      if (typeof parsed?.X === 'number' && typeof parsed?.O === 'number' && typeof parsed?.draws === 'number') {
        scores.value = parsed
      }
    } catch {}
  }
})

// persist scores
watch(scores, (val) => {
  localStorage.setItem('ttt_scores_v1', JSON.stringify(val))
}, { deep: true })

const isBoardFull = (b: Cell[]) => b.every(c => c !== null)

function evaluateWinner(b: Cell[]): { winner: Cell, line: number[] | null } {
  for (const line of WIN_LINES) {
    const [a, bIdx, c] = line
    if (b[a] && b[a] === b[bIdx] && b[a] === b[c]) {
      return { winner: b[a], line }
    }
  }
  return { winner: null, line: null }
}

const currentStatusText = computed(() => {
  if (status.value === 'in-progress') {
    return `Turn: ${currentPlayer.value}`
  }
  if (status.value === 'won' && winner.value) {
    return `Winner: ${winner.value}!`
  }
  return 'Draw!'
})

function resetBoard(swapFirst: boolean = false) {
  board.value = Array(9).fill(null)
  status.value = 'in-progress'
  winner.value = null
  winLine.value = null
  if (swapFirst) {
    currentPlayer.value = currentPlayer.value === 'X' ? 'O' : 'X'
  } else {
    currentPlayer.value = 'X'
  }
  // If bot should start
  if (mode.value === 'bot' && currentPlayer.value === 'O') {
    nextTick(() => botMove())
  }
}

// PUBLIC_INTERFACE
function newGame() {
  /** Start a new game and keep scores. */
  resetBoard(true)
}

// PUBLIC_INTERFACE
function restartSeries() {
  /** Reset board and scores. */
  scores.value = { X: 0, O: 0, draws: 0 }
  resetBoard(false)
}

function makeMove(index: number) {
  if (status.value !== 'in-progress') return
  if (board.value[index] !== null) return

  board.value[index] = currentPlayer.value
  const res = evaluateWinner(board.value)
  if (res.winner) {
    status.value = 'won'
    winner.value = res.winner
    winLine.value = res.line
    scores.value[res.winner] += 1
    return
  }
  if (isBoardFull(board.value)) {
    status.value = 'draw'
    scores.value.draws += 1
    return
  }

  // switch player
  currentPlayer.value = currentPlayer.value === 'X' ? 'O' : 'X'

  // if vs bot and it's bot's turn, trigger bot move
  if (mode.value === 'bot' && currentPlayer.value === 'O') {
    // small delay for UX
    setTimeout(() => botMove(), 250)
  }
}

function availableMoves(b: Cell[]): number[] {
  const arr: number[] = []
  for (let i = 0; i < 9; i++) if (b[i] === null) arr.push(i)
  return arr
}

// Easy: random
function botMoveEasy() {
  const moves = availableMoves(board.value)
  if (moves.length === 0) return
  const pick = moves[Math.floor(Math.random() * moves.length)]
  makeMove(pick)
}

// Medium: if can win, win; else if opponent can win next, block; else random center/corner/side
function findWinningMove(b: Cell[], player: 'X' | 'O'): number | null {
  for (const i of availableMoves(b)) {
    const clone = b.slice()
    clone[i] = player
    if (evaluateWinner(clone).winner === player) return i
  }
  return null
}

function botMoveMedium() {
  const b = board.value.slice()

  // try to win
  const winIdx = findWinningMove(b, 'O')
  if (winIdx !== null) return makeMove(winIdx)

  // block opponent
  const blockIdx = findWinningMove(b, 'X')
  if (blockIdx !== null) return makeMove(blockIdx)

  // heuristic: center, corners, sides
  const pref = [4, 0, 2, 6, 8, 1, 3, 5, 7]
  for (const i of pref) {
    if (b[i] === null) return makeMove(i)
  }
}

// Hard: minimax with alpha-beta pruning
function staticScore(b: Cell[]): number {
  const res = evaluateWinner(b)
  if (res.winner === 'O') return 10
  if (res.winner === 'X') return -10
  return 0
}

function minimax(b: Cell[], depth: number, isMax: boolean, alpha: number, beta: number): { score: number, move: number | null } {
  const score = staticScore(b)
  if (score !== 0 || isBoardFull(b) || depth === 0) return { score, move: null }

  const moves = availableMoves(b)

  if (isMax) {
    let best = { score: -Infinity, move: moves[0] ?? null }
    for (const m of moves) {
      b[m] = 'O'
      const res = minimax(b, depth - 1, false, alpha, beta)
      b[m] = null
      if (res.score > best.score) best = { score: res.score, move: m }
      alpha = Math.max(alpha, best.score)
      if (beta <= alpha) break
    }
    return best
  } else {
    let best = { score: Infinity, move: moves[0] ?? null }
    for (const m of moves) {
      b[m] = 'X'
      const res = minimax(b, depth - 1, true, alpha, beta)
      b[m] = null
      if (res.score < best.score) best = { score: res.score, move: m }
      beta = Math.min(beta, best.score)
      if (beta <= alpha) break
    }
    return best
  }
}

function botMoveHard() {
  // Depth cap to optimize. Full depth for 3x3 is fine; cap to remaining spaces.
  const remaining = availableMoves(board.value).length
  const depth = remaining // allow full search
  const b = board.value.slice()
  // If first move, prefer center
  if (remaining === 9 && b[4] === null) return makeMove(4)

  let best = { score: -Infinity, move: -1 as number }
  for (const m of availableMoves(b)) {
    b[m] = 'O'
    const res = minimax(b, depth - 1, false, -Infinity, Infinity)
    b[m] = null
    if (res.score > best.score) {
      best = { score: res.score, move: m }
    }
  }
  if (best.move !== -1) {
    makeMove(best.move)
  } else {
    botMoveMedium() // fallback
  }
}

function botMove() {
  if (status.value !== 'in-progress') return
  if (difficulty.value === 'easy') return botMoveEasy()
  if (difficulty.value === 'medium') return botMoveMedium()
  return botMoveHard()
}

function onModeChange() {
  resetBoard(false)
}

function onDifficultyChange() {
  if (mode.value === 'bot' && currentPlayer.value === 'O') {
    // restart so bot respects new difficulty from start
    resetBoard(false)
  }
}
</script>

<template>
  <section class="card">
    <div class="controls">
      <div class="selectors">
        <label>
          <span>Mode</span>
          <select v-model="mode" @change="onModeChange" aria-label="Game mode">
            <option value="pvp">Player vs Player</option>
            <option value="bot">Player vs Bot</option>
          </select>
        </label>

        <label :class="{ disabled: mode !== 'bot' }">
          <span>Difficulty</span>
          <select v-model="difficulty" :disabled="mode !== 'bot'" @change="onDifficultyChange" aria-label="Difficulty">
            <option value="easy">Easy</option>
            <option value="medium">Medium</option>
            <option value="hard">Hard</option>
          </select>
        </label>
      </div>

      <div class="actions">
        <button class="btn ghost" @click="restartSeries" title="Restart series and reset scores">
          Reset Scores
        </button>
        <button class="btn primary" @click="newGame" title="Start a new game">
          New Game
        </button>
      </div>
    </div>

    <div class="status">
      <span :class="[{ win: status==='won', draw: status==='draw' }]">{{ currentStatusText }}</span>
    </div>

    <div class="board" role="grid" aria-label="Tic Tac Toe Board">
      <button
        v-for="(cell, idx) in board"
        :key="idx"
        class="cell"
        role="gridcell"
        :aria-label="cell ? ('Cell ' + (idx+1) + ' ' + cell) : ('Cell ' + (idx+1) + ' empty')"
        :disabled="status !== 'in-progress' || (mode==='bot' && currentPlayer==='O') || !!cell"
        @click="makeMove(idx)"
        :data-highlight="winLine && winLine.includes(idx) ? 'true' : 'false'"
      >
        <span :class="cell === 'X' ? 'mark-x' : cell === 'O' ? 'mark-o' : ''">{{ cell }}</span>
      </button>
    </div>

    <div class="scoreboard">
      <div class="score x">
        <strong>X</strong>
        <span>{{ scores.X }}</span>
      </div>
      <div class="score draws">
        <strong>Draws</strong>
        <span>{{ scores.draws }}</span>
      </div>
      <div class="score o">
        <strong>O</strong>
        <span>{{ scores.O }}</span>
      </div>
    </div>
  </section>
</template>

<style scoped>
:root {
  --primary: #2563EB;
  --secondary: #F59E0B;
  --error: #EF4444;
  --background: #f9fafb;
  --surface: #ffffff;
  --text: #111827;
}

.card {
  width: 100%;
  max-width: 680px;
  padding: 20px;
  border-radius: 16px;
  background: var(--surface);
  color: var(--text);
  box-shadow: 0 10px 30px rgba(17,24,39,0.08), 0 4px 10px rgba(37,99,235,0.07);
  border: 1px solid rgba(17,24,39,0.06);
  transition: transform .2s ease, box-shadow .2s ease;
}
.card:hover {
  transform: translateY(-1px);
  box-shadow: 0 14px 38px rgba(17,24,39,0.10), 0 6px 16px rgba(37,99,235,0.09);
}

.controls {
  display: grid;
  grid-template-columns: 1fr auto;
  gap: 12px;
  align-items: end;
  margin-bottom: 14px;
}
@media (max-width: 640px) {
  .controls {
    grid-template-columns: 1fr;
  }
}

.selectors {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}
@media (max-width: 480px) {
  .selectors {
    grid-template-columns: 1fr;
  }
}

label {
  display: grid;
  gap: 6px;
  font-size: 0.85rem;
  color: rgba(17,24,39,0.8);
}
label.disabled {
  opacity: 0.6;
}
select {
  appearance: none;
  padding: 10px 12px;
  border-radius: 10px;
  border: 1px solid rgba(17,24,39,0.12);
  background:
    linear-gradient(180deg, rgba(59,130,246,0.08), rgba(255,255,255,0)) padding-box,
    linear-gradient(180deg, rgba(17,24,39,0.08), rgba(17,24,39,0.04)) border-box;
  color: var(--text);
  outline: none;
  transition: box-shadow .2s, border-color .2s;
}
select:focus {
  border-color: var(--primary);
  box-shadow: 0 0 0 4px rgba(37,99,235,0.15);
}

.actions {
  display: flex;
  gap: 10px;
  justify-content: end;
}
.btn {
  padding: 10px 14px;
  border: 1px solid rgba(17,24,39,0.12);
  border-radius: 10px;
  background: #fff;
  color: var(--text);
  cursor: pointer;
  transition: transform .12s ease, box-shadow .2s ease, background-color .2s;
}
.btn:hover {
  transform: translateY(-1px);
  box-shadow: 0 8px 22px rgba(17,24,39,0.08);
}
.btn.primary {
  background: linear-gradient(180deg, #2563EB, #1D4ED8);
  color: #fff;
  border-color: transparent;
}
.btn.primary:hover {
  box-shadow: 0 10px 24px rgba(37,99,235,0.28);
}
.btn.ghost {
  background: linear-gradient(180deg, rgba(245,158,11,0.12), rgba(255,255,255,0));
  border-color: rgba(245,158,11,0.35);
  color: #9A6700;
}

.status {
  text-align: center;
  margin: 6px 0 16px;
  font-weight: 600;
  color: var(--text);
}
.status .win {
  color: var(--secondary);
}
.status .draw {
  color: rgba(17,24,39,0.75);
}

.board {
  width: 100%;
  aspect-ratio: 1 / 1;
  max-width: 520px;
  margin: 0 auto 16px;
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 8px;
  background: linear-gradient(180deg, rgba(59,130,246,0.10), rgba(255,255,255,0));
  padding: 8px;
  border-radius: 16px;
  border: 1px solid rgba(17,24,39,0.06);
}

.cell {
  border: 1px solid rgba(17,24,39,0.12);
  background: var(--surface);
  border-radius: 14px;
  box-shadow: inset 0 2px 8px rgba(17,24,39,0.04);
  display: grid;
  place-items: center;
  font-size: clamp(2.4rem, 8vw, 4.5rem);
  font-weight: 800;
  color: var(--text);
  cursor: pointer;
  transition: transform .08s ease, box-shadow .2s ease, border-color .2s ease, background-color .2s ease;
}
.cell:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 8px 18px rgba(17,24,39,0.08);
  border-color: rgba(37,99,235,0.35);
}
.cell:disabled {
  cursor: not-allowed;
  opacity: 0.96;
}
.cell[data-highlight="true"] {
  background: linear-gradient(180deg, rgba(245,158,11,0.18), rgba(255,255,255,0.4));
  border-color: rgba(245,158,11,0.6);
  box-shadow: 0 10px 24px rgba(245,158,11,0.25);
}

.mark-x { color: var(--primary); text-shadow: 0 2px 10px rgba(37,99,235,0.25); }
.mark-o { color: var(--secondary); text-shadow: 0 2px 10px rgba(245,158,11,0.25); }

.scoreboard {
  margin-top: 10px;
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
}
.score {
  padding: 12px;
  background: #fff;
  border-radius: 12px;
  border: 1px solid rgba(17,24,39,0.08);
  box-shadow: 0 6px 16px rgba(17,24,39,0.06);
  display: grid;
  place-items: center;
  gap: 6px;
}
.score strong { font-size: 0.95rem; color: rgba(17,24,39,0.8); }
.score span { font-size: 1.25rem; font-weight: 800; color: var(--text); }
.score.x span { color: var(--primary); }
.score.o span { color: var(--secondary); }
.score.draws span { color: rgba(17,24,39,0.8); }
</style>
