<script setup lang="ts">
const url = ref('');
</script>

<template>
  <main class="h-screen w-screen">
    <VideoFlow :url="url" />
  </main>
</template>

<style scoped lang="scss"></style>
