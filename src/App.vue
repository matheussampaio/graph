<template>
  <network :nodes="nodes" :links="links">
  </network>
</template>

<script>
import Network from './components/Network.vue'

export default {
  components: {
    Network
  },
  data() {
    return {
      nodes: [
        { id: "12345" },
        { id: "67890" },
        { id: "45678" }
      ],
      links: [
        { source: "12345", target: "67890" },
        { source: "67890", target: "45678" },
        { source: "45678", target: "12345" }
      ]
    }
  }
}
</script>

<style>
</style>
