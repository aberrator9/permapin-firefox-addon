<template>
  <div class="container">

    <header class="flex items-center justify-center gap-2 mb-2 -translate-x-4">
      <Logo class="w-14"></Logo>
      <h1 class="text-5xl text-cyan-50">PinTabsPlus</h1>
    </header>

    <main>
      <div class="flex flex-col max-w-96 mx-auto my-4 space-y-4 items-center">
        <div class="inline-flex flex-initial items-center gap-x-2">
          <div class="text-cyan-50">Pin tabs to:</div>
          <select v-model="pinTo"
            class="p-1 bg-cyan-900 hover:bg-cyan-700 text-cyan-50 border-2 border-cyan-300 rounded-md">
            <option>respective windows</option>
            <option>all windows</option>
            <option>main window</option>
          </select>
          <select v-show="pinTo === 'main window'" disabled v-model="mainWindow">
            <!-- <div v-for="" >
            </div> -->
          </select>
        </div>
        <div class="inline-flex flex-initial items-center gap-x-2">
          <input type="checkbox" v-model="keepAlive"
            class="bg-cyan-900 text-cyan-50 border-2 border-cyan-300 rounded-md" />
          <p class="text-cyan-50">When closing a window with pinned tabs, move tabs to another window</p>
        </div>
        <div class="inline-flex flex-initial items-center gap-x-2">
          <input type="checkbox" v-model="autoSave"
            class="bg-cyan-900 text-cyan-50 border-2 border-cyan-300 rounded-md" />
          <p class="text-cyan-50">Automatically update pinned tabs</p>
        </div>
        <button @click="savePins()" :disabled="autoSave" :class="{
          'bg-cyan-900 hover:bg-cyan-700 active:bg-cyan-800 text-cyan-50 border-2 border-cyan-300 rounded-md px-2 py-1': !autoSave,
          'bg-gray-600 text-gray-300 border-2 border-gray-500 rounded-md px-2 py-1': autoSave
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

const mainWindow = ref(-1)
const pinnedTabs = ref([])
const autoSave = ref(false)
const keepAlive = ref(false)
const pinTo = ref('')

const loadPins = async (storedTabs) => {
  console.log('storedTabs', storedTabs)
  const currentPins = await browser.tabs.query({ pinned: true })
  const pinnedTabsIdSet = new Set(currentPins.map((x) => x.id))

  const missingTabs = Object.values(storedTabs).filter((x) => !pinnedTabsIdSet.has(x.id))
  console.log('missingTabs', missingTabs)
  if (missingTabs.length <= 0) {
    return
  }

  if (pinTo === 'main window') {
    // Determine main window - one with most pinned tabs?
    // User can select
    // Move all current tabs there, ideally without changing the windowId in the stored data
  }
  else if (pinTo === 'all windows') {
    // Remove all pinned tabs and replace them with all stored tabs
  }
  else {  // Pin to respective windows
    const windowIdsSet = new Set(Object.values(await browser.windows.getAll()).map((x) => x.id))
    const newWindowMap = new Map()

    for (let tab of missingTabs) {
      if (!windowIdsSet.has(tab.windowId) && !newWindowMap.has(tab.windowId)) {
        const newWindow = await browser.windows.create()
        console.log('Creating missing window', newWindow.id)
        newWindowMap.set(tab.windowId, newWindow.id)
      }

      await browser.tabs.create({
        active: false,
        pinned: true,
        url: tab.url,
        windowId: newWindowMap.has(tab.windowId) ? newWindowMap.get(tab.windowId) : tab.windowId
      })
    }

  }
}

const savePins = async () => {
  const pinnedTabs = await browser.tabs.query({ pinned: true })
  console.log('Saving pins:', pinnedTabs)

  localStorage.setItem('pinnedTabs', JSON.stringify(pinnedTabs))
}

onMounted(() => {
  const storedPinTo = localStorage.getItem('pinTo')
  pinTo.value = storedPinTo ? storedPinTo : 'respective windows'
  const storedSync = localStorage.getItem('syncPins')
  autoSave.value = storedSync ? storedSync : false

  pinnedTabs.value = JSON.parse(localStorage.getItem('pinnedTabs'))
  if (pinnedTabs) {
    try {
      loadPins(pinnedTabs.value)
    } catch (e) {
      console.error(e)
    }
  } else {
    savePins()
  }
})

const addPin = async (tabId) => {
  console.log('Adding', tabId)
  pinnedTabs.value.push(await browser.tabs.query({ index: tabId }))

  if (autoSave.value) {
    savePins()
  }
}

const removePin = async (tabId) => {
  console.log('Removing', tabId)
  pinnedTabs.value = pinnedTabs.value.filter((x) => x.id !== tabId)

  if (autoSave.value) {
    savePins()
  }
}

const onTabPinned = (tabId, props) => {
  if (props['pinned']) {
    addPin(tabId)
  } else {
    removePin(tabId)
  }
}

const onTabRemoved = (tabId, removeInfo) => { // removeInfo properties: windowId, isWindowClosing
  if (removeInfo.isWindowClosing) {
    return;
  }
  removePin(tabId)
}

const onTabMoved = (tabId, moveInfo) => { // moveInfo properties: windowId, fromIndex, toIndex
  console.log(`Tab ${tabId} moved from ${moveInfo.fromIndex} to ${moveInfo.toIndex}, windowId: ${moveInfo.windowId}`)
}

browser.tabs.onUpdated.addListener(
  onTabPinned, { properties: ['pinned'] }
)

browser.tabs.onRemoved.addListener(
  onTabRemoved
)

browser.tabs.onMoved.addListener(
  onTabMoved
)

watch(autoSave, (val) => {
  console.log('syncPins ->', val)
  localStorage.setItem('syncPins', val)

  if (val === true) {
    savePins()
  }
})

watch(pinTo, (val) => {
  console.log('pinTo ->', val)
  localStorage.setItem('pinTo', val)
})

</script>