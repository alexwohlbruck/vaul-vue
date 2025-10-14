<script setup lang="ts">
import { computed, ref, watchEffect } from 'vue'
import { DialogContent } from 'reka-ui'
import { injectDrawerRootContext } from './context'
import { useScaleBackground } from './useScaleBackground'

const {
  open,
  isOpen,
  snapPointsOffset,
  snapPoints,
  activeSnapPoint,
  hasSnapPoints,
  drawerRef,
  onPress,
  onDrag,
  onRelease,
  modal,
  emitOpenChange,
  dismissible,
  keyboardIsOpen,
  closeDrawer,
  direction,
  handleOnly,
} = injectDrawerRootContext()

useScaleBackground()

const delayedSnapPoints = ref(false)

const snapPointHeight = computed(() => {
  // Find the index of the active snap point
  const activeIndex = snapPoints.value?.findIndex(sp => sp === activeSnapPoint.value) ?? 0
  
  if (snapPointsOffset.value && snapPointsOffset.value.length > 0) {
    const index = activeIndex >= 0 ? activeIndex : 0
    return `${snapPointsOffset.value[index]}px`
  }

  return '0'
})

function handlePointerDownOutside(event: Event) {
  if (!modal.value || event.defaultPrevented) {
    event.preventDefault()
    return
  }
  if (keyboardIsOpen.value)
    keyboardIsOpen.value = false

  if (dismissible.value) {
    emitOpenChange(false)
  }
  else {
    event.preventDefault()
  }
}

function handlePointerDown(event: PointerEvent) {
  if (handleOnly.value)
    return

  onPress(event)
}

function handleOnDrag(event: PointerEvent) {
  if (handleOnly.value)
    return

  onDrag(event)
}

watchEffect (() => {
  if (hasSnapPoints.value) {
    window.requestAnimationFrame(() => {
      delayedSnapPoints.value = true
    })
  }
})
</script>

<template>
  <DialogContent
    ref="drawerRef"
    data-vaul-drawer=""
    :data-vaul-drawer-direction="direction"
    :data-vaul-delayed-snap-points="delayedSnapPoints ? 'true' : 'false'"
    :data-vaul-snap-points="isOpen && hasSnapPoints ? 'true' : 'false'"
    :style="{ '--snap-point-height': snapPointHeight }"
    @pointerdown="handlePointerDown"
    @pointermove="handleOnDrag"
    @pointerup="onRelease"
    @pointer-down-outside="handlePointerDownOutside"
    @open-auto-focus.prevent
    @escape-key-down="(event) => {
      if (!dismissible)
        event.preventDefault()
    }"
  >
    <slot />
  </DialogContent>
</template>
