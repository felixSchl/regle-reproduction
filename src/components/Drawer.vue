<template>
  <Teleport to="body">
    <div ref="drawer" class="drawer fixed bottom-0 h-[500px] bg-white z-10 w-full">
      <div ref="handle" class="h-[4px] w-full bg-gray-200 cursor-row-resize"></div>
      <slot />
    </div>
  </Teleport>
</template>

<script setup lang="ts">
import { useDraggable } from '@vueuse/core'
import { computed, useTemplateRef } from 'vue'

const handle = useTemplateRef('handle')

const { position } = useDraggable(handle, {
  axis: 'y',
  initialValue: {
    y: document.documentElement.clientHeight - 500,
    x: 0,
  },
})

const drawerHeight = computed(() => {
  return `${document.documentElement.clientHeight - position.value.y}px`
})
</script>

<style lang="scss" scoped>
.drawer {
  height: v-bind(drawerHeight);
  box-shadow: -10px 0 15px rgba(0, 0, 0, 0.1);
  overflow-y: scroll;
}
</style>
