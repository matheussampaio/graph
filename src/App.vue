<template>
  <div>
    <div>
      <input type="text" v-model="source" placeholder="source">
      <input type="text" v-model="target" placeholder="target">
      <input type="number" v-model="timeToClose" placeholder="target">
      <select v-model="size">
        <option :value="'small'">Small</option>
        <option :value="'medium'">Medium</option>
        <option :value="'large'">Large</option>
      </select>
      <button @click="addLink">Add Link</button>
      <button @click="clear">Clear</button>
    </div>
    <network :nodes="nodes" :links="links" :width="width" :height="height"></network>
  </div>
</template>

<script>
import moment from 'moment'

import Network from './components/Network.vue'

export default {
  name: 'App',
  components: {
    Network
  },
  data() {
    return {
      nodes: [
      ],
      links: [
      ],
      source: '',
      target: '',
      timeToClose: 5,
      size: 'small',
      interval: null,
      width: window.innerWidth - 30,
      height: window.innerHeight - 100,
    }
  },
  methods: {
    clear() {
      this.nodes = []
      this.links = []

      localStorage.removeItem('nodes')
      localStorage.removeItem('links')
    },
    addLink() {
      const sourceExists = this.nodes.find(n => n.id === this.source)

      if (!sourceExists) {
        this.nodes.push({ id: this.source })
      }

      const targetExists = this.nodes.find(n => n.id === this.target)

      if (!targetExists) {
        this.nodes.push({ id: this.target })
      }

      if (!sourceExists || !targetExists) {
        localStorage.setItem("nodes", JSON.stringify(this.nodes))
      }

      this.links = this.links.filter(l => l.source !== this.source || l.target !== this.target)

      let color = 'green'

      if (this.size === 'small') {
        color = 'green' 
      }

      if (this.size === 'medium') {
        color = 'blue' 
      }

      if (this.size === 'large') {
        color = 'yellow' 
      }

      this.links.push({
        color,
        source: this.source,
        target: this.target,
        timeToClose: moment().add(this.timeToClose, 'hours').toISOString()
      })

      localStorage.setItem("links", JSON.stringify(this.links))
    },
    update() {
      const now = moment.now()

      this.links = this.links.filter(l => moment(l.timeToClose).isAfter(now))

      localStorage.setItem("links", JSON.stringify(this.links))
    },
    onResize() {
      this.height = window.innerHeight - 100
      this.width = window.innerWidth - 30
    }
  },
  mounted() {
    if (localStorage.getItem('links')) {
      this.links = JSON.parse(localStorage.getItem('links'));
    }

    if (localStorage.getItem('nodes')) {
      this.nodes = JSON.parse(localStorage.getItem('nodes'));
    }

    if (this.interval == null) {
      this.update()

      this.interval = setInterval(() => this.update(), 5 * 60 * 1000)
    }

    this.$nextTick(() => {
      window.addEventListener('resize', this.onResize);
    })
  },
  beforeDestroy() {
    if (this.interval) {
      clearInterval(this.interval)

      this.interval = null
    }

    window.removeEventListener('resize', this.onResize); 
  }
}
</script>

<style>
</style>
