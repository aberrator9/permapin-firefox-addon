<template>
  <div class="container">

    <header class="flex items-center justify-center gap-2 mb-2 -translate-x-4">
      <Logo class="w-14"></Logo>
      <h1 class="text-5xl text-cyan-50">PermaPin</h1>
    </header>

    <main>

      <!-- <div class="flex flex-col gap-4 items-center">
        <label for="urls" class="text-cyan-50 text-lg">Urls (separate with commas or new line):</label>
        <textarea v-model="urls" @blur="parseUrls()" name="urls"
          class="flex-auto resize-none border-cyan-600 border-2 bg-cyan-50 rounded-md break-all w-96 h-64">
        </textarea>
      </div> -->

      <div class="flex flex-col max-w-96 mx-auto my-4 space-y-4 items-center">
        <div class="inline-flex flex-initial items-center gap-x-2">
          <div class="text-cyan-50">Pin tabs to:</div>
          <select v-model="pinTo" class="p-1 bg-cyan-900 hover:bg-cyan-700 text-cyan-50 border-2 border-cyan-300 rounded-md">
            <option>respective windows</option>
            <option>main window</option>
            <option>all windows</option>
          </select>
        </div>
        <div class="inline-flex flex-initial items-center gap-x-2">
          <input type="checkbox" v-model="syncPinnedTabs"
          class="bg-cyan-900 text-cyan-50 border-2 border-cyan-300 rounded-md" />
          <p class="text-cyan-50">Automatically update pinned tabs</p>
        </div>
        <button @click="storePins()" :disabled="syncPinnedTabs"
          :class="{
            'bg-cyan-900 hover:bg-cyan-700 active:bg-cyan-800 text-cyan-50 border-2 border-cyan-300 rounded-md px-2 py-1': !syncPinnedTabs,
            'bg-gray-600 text-gray-300 border-2 border-gray-500 rounded-md px-2 py-1': syncPinnedTabs
          }">
          Update pinned tabs now
        </button>
      </div>

    </main>

  </div>
</template>

<script setup>

import { onMounted, ref, watch } from 'vue'
import Logo from './components/Logo.vue'

let pinnedTabs = []
const syncPinnedTabs = ref(false)
const pinTo = ref('respective windows')

// const getUrls = (tabs) => {
//   return tabs.map((t) => t.url).join('\n')
// }

const retrievePins = async (storedPins) => {
  const storedPinsArr = JSON.parse(storedPins)
  
  pinnedTabs = await browser.tabs.query({ pinned: true })
  const pinnedTabUrlsSet = new Set(pinnedTabs.map((x) => x.url))

  const missingTabs = storedPinsArr.filter((x) => !pinnedTabUrlsSet.has(x.url))
  
  missingTabs.forEach(tab => {
      const newTab = browser.tabs.create({
        pinned: true,
        url: tab.url,
        windowId: tab.windowId
      })
  });
}

const storePins = async () => {
  pinnedTabs = await browser.tabs.query({ pinned: true })
  console.log(pinnedTabs)
  
  localStorage.setItem('pinnedTabs', JSON.stringify(pinnedTabs))
}

onMounted(() => {
  const storedPins = localStorage.getItem('pinnedTabs')
  if (storedPins) {
    try {
      retrievePins(storedPins)
    } catch (e) {
      console.error(e);
    }
  } else {
    storePins()
  }
})

watch(pinnedTabs.value, () => {
  if(syncPinnedTabs) {
    localStorage.setItem('pinnedTabs', JSON.stringify(pinnedTabs))
  }
},
  { deep: true }
);


</script>