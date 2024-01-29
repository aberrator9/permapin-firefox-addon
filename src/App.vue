<script setup>
import Logo from './components/Logo.vue';


</script>

<template>
  <header class="flex place-items-center">
    <Logo class="flex-auto"></Logo>
    <h1 class="flex-auto text-4xl font-bold text-cyan-50">PermaPin</h1>
  </header>

  <main class="flex place-items-center">
    <div class="flex-wrap container place-items-start">
      <label for="urls" class="text-cyan-50">Urls (separate with commas or new line):</label>
      <textarea name="urls" class="flex-auto resize-none bg-cyan-50 font-medium text-lg m-4 w-96"></textarea>
    </div>
  </main>
</template>