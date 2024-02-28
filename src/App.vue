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
          <input type="checkbox" v-model="syncPins"
            class="bg-cyan-900 text-cyan-50 border-2 border-cyan-300 rounded-md" />
          <p class="text-cyan-50">Automatically update pinned tabs</p>
        </div>
        <button @click="savePins()" :disabled="syncPins" :class="{
          'bg-cyan-900 hover:bg-cyan-700 active:bg-cyan-800 text-cyan-50 border-2 border-cyan-300 rounded-md px-2 py-1': !syncPins,
          'bg-gray-600 text-gray-300 border-2 border-gray-500 rounded-md px-2 py-1': syncPins
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

const pinnedTabs = ref([])
const syncPins = ref(false)
const keepAlive = ref(false)
const pinTo = ref('')

const loadPins = async (storedTabs) => {
  const currentPins = await browser.tabs.query({ pinned: true })
  const pinnedTabUrlsSet = new Set(currentPins.map((x) => x.url))

  const missingTabs = storedTabs.filter((x) => !pinnedTabUrlsSet.has(x.url))
  if (!missingTabs.length) {
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
    const storedTabsWindowIdSet = new Set(Object.values(storedTabs).map((x) => x.windowId))

    const tabsWithMissingWindows = missingTabs.filter((t) => !storedTabsWindowIdSet.has(t.windowId))
    tabsWithMissingWindows.sort((a, b) => a.windowId - b.windowId);

    const tabsByWindowId = {};

    for (const tab of tabsWithMissingWindows) {
      if (!tabsByWindowId[tab.windowId]) {
        tabsByWindowId[tab.windowId] = [];
      }
      tabsByWindowId[tab.windowId].push(tab);
    }

    const groupedTabs = Object.values(tabsByWindowId);

    for (const tabGroup of groupedTabs) {
      console.log('tabGroup urls in', tabGroup[0].windowId, ':', tabGroup.map((t) => t.url))
      // const newWindow = browser.windows.create()

      // Below creates infinite tabs. Maybe due to the app page being opened repeatedly
      // for (const tab in tabGroup) {
      //   browser.tabs.create({
      //     active: false,
      //     pinned: true,
      //     url: tab.url,
      //     windowId: newWindow.id
      //   })
      // }
    }

    for (const tab of missingTabs) {
      console.log(tab.url, 'is missing; creating...')
      browser.tabs.create({
        active: false,
        pinned: true,
        url: tab.url,
        windowId: mainWindow === -1 ? tab.windowId : mainWindow
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
  syncPins.value = storedSync ? storedSync : false

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

  if (syncPins.value) {
    savePins()
  }
}

const removePin = async (tabId) => {
  console.log('Removing', tabId)
  pinnedTabs.value = pinnedTabs.value.filter((x) => x.id !== tabId)

  if (syncPins.value) {
    savePins()
  }
}

const onTabPinned = (tabId, props) => {
  if (props['pinned']) {
    addPin(tabId)
  } else {
    removePin(tabId)
  }

  if (syncPins.value) {
    savePins()
  }
}
                          
const onTabRemoved = (tabId, removeInfo) => { // removeInfo properties: windowId, isWindowClosing
  removePin(tabId)
}

const onTabMoved = (tabId, moveInfo) => { // moveInfo properties: windowId, fromIndex, toIndex

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

watch(syncPins, (val) => {
  console.log('syncPins ->', val)
  localStorage.setItem('syncPins', val)

  if(val === true) {
    savePins()
  }
})

watch(pinTo, (val) => {
  console.log('pinTo ->', val)
  localStorage.setItem('pinTo', val)
})

</script>