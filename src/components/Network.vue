<template>
</template>

<script>
import { Network } from 'vue-visjs'

export default {
  components: {
    Network
  },
  props: {
    nodes: {
      type: Array,
      default: () => []
    },
    edges: {
      type: Array,
      default: () => []
    },
    options: {
      type: Object,
      default: () => ({})
    },
  }
}
</script>

<style scoped>
.network {
  height: 100vh;
}
</style>
