<script setup lang="ts">
import TicTacToe from '@/components/TicTacToe.vue'
</script>

<template>
  <main class="home">
    <TicTacToe />
  </main>
</template>

<style scoped>
.home {
  width: 100%;
  display: grid;
  place-items: center;
  padding: 16px;
}
</style>
