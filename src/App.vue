<template>
  <network
    class="network"
    :nodes="nodes"
    :edges="edges"
    :options="options"
    >
  </network>
</template>

<script>
import { Network } from 'vue-visjs'

export default {
  name: 'App',
  components: {
    Network
  },
  data() {
    return {
      id: 0,
      label: '',
      from: 0,
      to: 0,
      nodes: [
      ],
      edges: [
      ]
    }
  },
  methods: {
    addNode() {
      this.nodes.push({ id: this.id, label: this.label })
    },
    addEdge() {
      this.edges.push({ from: this.from, to: this.to })
    }
  },
  computed: {
    options() {
      return {
        autoResize: true,
        height: '100%',
        width: '100%',
        manipulation: {
          enabled: true
        }
      }
    }
  }
}
</script>

<style>
#app {
  height: 100vh;
}

.network {
  height: 100vh;
}
</style>
