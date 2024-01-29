<script setup>

import { ref } from 'vue'
import Logo from './components/Logo.vue'

const tabsText = ref('')
const tabs = []


</script>

<template>
  <div class="container">

    <header class="flex items-center justify-center gap-2 mb-2 -translate-x-4">
      <Logo class="w-14"></Logo>
      <h1 class="text-5xl text-cyan-50">PermaPin</h1>
    </header>

    <main>
      <div class="flex flex-col gap-4 items-center">
        <label for="urls" class="text-cyan-50 text-lg">Urls (separate with commas or new line):</label>
        <textarea v-bind="tabsText" @input="console.log(tabsText)" name="urls"
          class="flex-auto resize-none border-cyan-600 border-2 bg-cyan-50 font-medium text-lg rounded-md w-96"></textarea>
      </div>

      <div class="flex justify-self-center justify-between max-w-96 mx-auto my-4">
        <button class="bg-cyan-900 hover:bg-cyan-700 active:bg-cyan-800 text-cyan-50 border-2 border-cyan-300 rounded-md px-2 py-1">Use currently pinned</button>
        <div class="inline-flex flex-initial items-center gap-x-2">
          <Input type="checkbox" class="bg-cyan-900 text-cyan-50 border-2 border-cyan-300 rounded-md"></Input>
          <p class="text-cyan-50">Pin to all windows</p>
        </div>
      </div>
    </main>

  </div>
</template>