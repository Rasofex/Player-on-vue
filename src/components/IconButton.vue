<template>
  <button class="rounded-full text-zinc-400 hover:text-black dark:hover:text-white transition-all">
    <div class="flex justify-center items-center w-full h-full">
      <slot></slot>
    </div>
  </button>
</template>

<script>
import { defineComponent } from 'vue'

export default defineComponent({
  name: 'IconButton'
})
</script>

<style scoped></style>
