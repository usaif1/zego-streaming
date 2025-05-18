<script setup lang="ts">
import ZegoLiveStream from './components/ZegoLiveStream.vue'
</script>

<template>
  <div class="app-container">
    <main>
      <ZegoLiveStream />
    </main>
  </div>
</template>

<style scoped>
.app-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem;
}
</style>
