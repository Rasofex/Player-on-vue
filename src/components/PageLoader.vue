<template>
  <div class="fixed z-50 w-screen h-screen bg-white dark:bg-zinc-800">
    <div class="flex justify-center items-center w-full h-full dark:text-white">
      <Icon size="36">
        <LoadingOutlined class="animate-spin" />
      </Icon>
    </div>
  </div>
</template>

<script>
import { defineComponent } from 'vue'
import { Icon } from '@vicons/utils'
import { LoadingOutlined } from '@vicons/antd'

export default defineComponent({
  name: 'PageLoader',
  components: {
    Icon,
    LoadingOutlined
  }
})
</script>

<style scoped></style>
