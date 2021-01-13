<template>
  <div>
    <svg id="svg">
    </svg>
  </div>
</template>

<script>
import moment from 'moment'
import * as d3 from "d3"

export default {
  props: {
    nodes: {
      type: Array,
      default: () => []
    },
    links: {
      type: Array,
      default: () => []
    },
    height: {
      type: Number,
      default: 500
    },
    width: {
      type: Number,
      default: 500
    }
  },
  data() {
    return {
      interval: null
    }
  },
  computed: {
    graph() {
      const chart = (() => {
        const svg = d3.select("#svg")
          .attr("width", '100%')
          .attr("height", '100%')
          .attr("viewBox", [-this.width / 2, -this.height / 2, this.width, this.height])
          .call(d3.zoom().on("zoom", () => {
            console.log('zoom')
            svg.attr("transform", d3.event.transform)
          }))
        
        const container = svg.append("g")
          .attr("id", "container")
          .call(d3.zoom().on("zoom", () => {
            console.log('zoom')
            container.attr("transform", d3.event.transform)
          }))

        const simulation = d3.forceSimulation()
          .force("charge", d3.forceManyBody().strength(-1000))
          .force("link", d3.forceLink().id(d => d.id).distance(200))
          .force("x", d3.forceX())
          .force("y", d3.forceY())
          .on("tick", ticked)

        let link = container.append('g')
          .selectAll(".link")
          .append("line")
          .attr("class", "link")
        
        let linkLabel = container.append('g')
          .selectAll('.link-label')
          .append('text')
          .attr("class", "link-label")

        let node = container.append('g')
          .selectAll('.node')
          .append('g')
          .attr("class", "node")
          
        function ticked() {
          link.attr("x1", d => d.source.x)
            .attr("y1", d => d.source.y)
            .attr("x2", d => d.target.x)
            .attr("y2", d => d.target.y)

          linkLabel.attr("transform", d => `translate(${(d.source.x + d.target.x) / 2}, ${(d.source.y + d.target.y) / 2})`)
          node.attr("transform", d => `translate(${d.x},${d.y})`)
        }

        return Object.assign(svg.node(), {
          update({ nodes, links }) {
            // Make a shallow copy to protect against mutation, while
            // recycling old nodes to preserve position and velocity.
            const old = new Map(node.data().map(d => [d.id, d]))

            nodes = nodes.map(d => Object.assign(old.get(d.id) || {}, d))
            links = links.map(d => Object.assign({}, d))

            node = node
              .data(nodes, d => d.id)
              .join(enter => {
                const group = enter.append('g')
                  .attr('class', 'node')

                group.append("circle")
                    .attr("r", 8)
                    .attr("fill", d => d.color || 'black')

                group.append("text")
                    .attr("dx", d => (d.id.length / 2) * -10)
                    .attr("dy", 25)
                    .attr('fill', d => d.textColor || 'black') 
                    .text(d => d.id)

                return group
              })

            link = link
              .data(links, d => [d.source, d.target])
              .join(enter => enter
                .append('line')
                .attr('class', 'link')
                .attr('stroke', d => d.color || 'black') 
                .attr('id', d => `${d.source}->${d.target}`)
              )
            
            linkLabel = linkLabel
              .data(links, d => [d.source, d.target])
              .join(enter => enter
                .append('text')
                .attr('class', 'link-label')
                .text(d => moment(d.timeToClose).fromNow())
              )

            simulation.nodes(nodes)
            simulation.force("link").links(links)
            simulation.alpha(1).restart()
          }
        })
      })()

      return chart
    },
  },
  methods: {
    update() {
      console.log('updating graph')

      this.graph.update({
        nodes: this.nodes,
        links: this.links
      })
    }
  },
  watch: {
    nodes() {
      this.update()
    },
    links() {
      this.update()
    }
  },
  mounted() {
    if (this.interval == null) {
      this.interval = setInterval(() => this.update(), 5 * 60 * 1000)
    }
  },
  destoyed() {
    if (this.interval) {
      clearInterval(this.interval)

      this.interval = null
    }
  }
}
</script>

<style>
svg {
  border: 1px solid black;
}

.node text {
  pointer-events: none;
  font-size: 20px;
}
</style>
