<script setup lang="ts">
import TicTacToe from './components/TicTacToe.vue'
</script>

<template>
  <div class="app-wrap">
    <header class="app-header">
      <div class="brand">
        <div class="logo">◎</div>
        <div class="titles">
          <h1>Tic Tac Toe</h1>
          <p>Ocean Professional</p>
        </div>
      </div>
    </header>

    <main class="content">
      <TicTacToe />
    </main>

    <footer class="app-footer">
      <span>Made with Vue 3 + Vite</span>
    </footer>
  </div>
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

.app-wrap {
  min-height: 100vh;
  display: grid;
  grid-template-rows: auto 1fr auto;
  background: linear-gradient(180deg, rgba(59,130,246,0.08), rgba(249,250,251,1));
}

.app-header {
  background: var(--surface);
  border-bottom: 1px solid rgba(17,24,39,0.06);
  position: sticky;
  top: 0;
  z-index: 10;
}

.brand {
  max-width: 1100px;
  margin: 0 auto;
  padding: 1rem 1.25rem;
  display: flex;
  align-items: center;
  gap: 12px;
}

.logo {
  width: 40px;
  height: 40px;
  border-radius: 12px;
  background: radial-gradient(circle at 30% 30%, rgba(37,99,235,0.9), rgba(37,99,235,0.6));
  color: white;
  display: grid;
  place-items: center;
  font-weight: 800;
  box-shadow: 0 6px 16px rgba(37,99,235,0.25);
}

.titles h1 {
  margin: 0;
  font-size: 1.25rem;
  color: var(--text);
}
.titles p {
  margin: 0;
  font-size: 0.85rem;
  color: rgba(17,24,39,0.6);
}

.content {
  display: grid;
  place-items: center;
  padding: 24px 16px;
}

.app-footer {
  padding: 0.75rem 1rem;
  text-align: center;
  color: rgba(17,24,39,0.6);
  font-size: 0.9rem;
  border-top: 1px solid rgba(17,24,39,0.06);
  background: var(--surface);
}
</style>
