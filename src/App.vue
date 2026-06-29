<template>
  <div id="app">
    <div>
      <navbar />

      <main class="main-content">
        <RouterView />
      </main>
    </div>
  </div>
</template>

<script setup lang="ts">
import { RouterView } from "vue-router";
import navbar from "@/components/navbar.vue";
</script>

<style scoped lang="scss">
#app {
  position: relative;
  min-height: 100vh;
}


</style>
