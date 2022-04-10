<template>
  <q-layout view="hHh lpR fFf">
    <q-drawer
      id="layout-drawer"
      v-model="drawer"
      show-if-above
      :width="100"
      :breakpoint="400"
      style="border-right: 1px solid #ddd"
    >
      <q-img id="yeah" class="absolute-top" style="height: 150px">
        <div class="absolute-center bg-transparent">
          <q-avatar size="100px" class="q-mb-sm">
            <img src="https://cdn.glitch.global/bdb630b5-11ba-4505-a157-8b517abc9ed1/EatEthDesign%20(2).png?v=1649622094358" />
          </q-avatar>
        </div>
      </q-img>
    </q-drawer>



    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import { ref } from 'vue'

export default {
  setup () {
    const leftDrawerOpen = ref(false)

    return {
      leftDrawerOpen,
      toggleLeftDrawer () {
        leftDrawerOpen.value = !leftDrawerOpen.value
      }
    }
  }
}
</script>