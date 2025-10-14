<script setup lang="ts">
import { computed, ref } from 'vue';
import { useScroll } from '@vueuse/core'
import { DrawerContent, DrawerHandle, DrawerOverlay, DrawerPortal, DrawerRoot, DrawerTrigger } from 'vaul-vue'

const snapPoints = ref(['180px', 0.5, 1])
const scrollContainer = ref<HTMLElement | null>(null)
const activeSnapPoint = ref<number | string | null>(null)

let lastTouchY = 0
let isScrollingUp = false
const { y: scrollY } = useScroll(scrollContainer)
const isAtTop = computed(() => scrollY.value === 0)
const isFullyExpanded = computed(() => activeSnapPoint.value === 1)

function snapPointChanged(snapPoint: number | string | null) {
  activeSnapPoint.value = snapPoint
}

function handleTouchStart(e: TouchEvent) {
  lastTouchY = e.touches[0].clientY
}

function handleTouchMove(e: TouchEvent) {
  if (!isFullyExpanded.value) return
  
  const currentTouchY = e.touches[0].clientY
  const deltaY = currentTouchY - lastTouchY
  isScrollingUp = deltaY > 0 // Positive deltaY means scrolling up (finger moving down)
  
  // Prevent upward scroll when at the top of the container
  if (isAtTop.value && isScrollingUp) {
    e.preventDefault()
    return false
  }
  
  lastTouchY = currentTouchY
}

function handleTouchEnd() {
  isScrollingUp = false
}

const isOpen = ref(false)
</script>

<template>
  <DrawerRoot
    v-model:open="isOpen"
    direction="bottom"
    :snap-points="snapPoints"
    v-model:active-snap-point="activeSnapPoint"
    @update:activeSnapPoint="snapPointChanged"
    :default-snap-point="2"
    :modal="false"
    :repositionInputs="true"
    :dismissible="true"
    >
    <DrawerTrigger
      class="rounded-full bg-white px-4 py-2.5 text-sm font-semibold text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 hover:bg-gray-50"
    >
      Open Drawer
    </DrawerTrigger>
    <DrawerPortal>
      <DrawerOverlay class="fixed bg-black/40 inset-0" />
      <DrawerContent
        class="bg-gray-100 flex flex-col rounded-t-[10px] h-full fixed bottom-0 left-0 right-0 border-t"
      >
        <div
          ref="scrollContainer"
          class="p-4 bg-white rounded-t-[10px] flex-1"
          :class="{ 'overflow-y-auto': isFullyExpanded }"
          :style="{ touchAction: isFullyExpanded ? 'pan-y' : 'none' }"
          @touchstart="handleTouchStart"
          @touchmove="handleTouchMove"
          @touchend="handleTouchEnd"
          >

          <div class="max-w-md mx-auto">
            <h2 id="radix-:R3emdaH1:" class="font-medium mb-4">
              Drawer for Vue.
            </h2>

            <button @click="isOpen = false">Close</button>

            <div class="overflow-x-auto -mx-4 px-4 mb-8 touch-pan-x">
              <div class="flex gap-4">
                <div class="w-[200px] h-[150px] flex-shrink-0 bg-gray-300 rounded-lg"></div>
                <div class="w-[200px] h-[150px] flex-shrink-0 bg-gray-300 rounded-lg"></div>
                <div class="w-[200px] h-[150px] flex-shrink-0 bg-gray-300 rounded-lg"></div>
                <div class="w-[200px] h-[150px] flex-shrink-0 bg-gray-300 rounded-lg"></div>
                <div class="w-[200px] h-[150px] flex-shrink-0 bg-gray-300 rounded-lg"></div>
              </div>
            </div>

            <p v-for="i in 100" :key="i">
              test {{ i }}
            </p>
          </div>
        </div>
      </DrawerContent>
    </DrawerPortal>
  </DrawerRoot>
</template>

<style scoped>
[data-vaul-drawer] {
  position: fixed;
}

[data-vaul-drawer][data-vaul-snap-points=true][data-vaul-drawer-direction=bottom][data-state=open] {
  animation-name: slideFromBottomToSnapPoint;
}

@keyframes slideFromBottomToSnapPoint {
  from {
    transform: translate3d(0, var(--initial-transform, 100%), 0);
  }
  to {
    transform: translate3d(0, var(--snap-point-height, 1), 0);
  }
}
</style>
