<template>
  <div>
    <svg id="svg">
    </svg>
  </div>
</template>

<script>
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
    }
  },
  computed: {
    chart() {
      const chart = (() => {
        const height = 500
        const width = 500

        const color = d3.scaleOrdinal(d3.schemeTableau10)

        const svg = d3.select("#svg")
          .attr("width", width)
          .attr("height", height)
          .attr("viewBox", [-width / 2, -height / 2, width, height]);

        const simulation = d3.forceSimulation()
          .force("charge", d3.forceManyBody().strength(-1000))
          .force("link", d3.forceLink().id(d => d.id).distance(200))
          .force("x", d3.forceX())
          .force("y", d3.forceY())
          .on("tick", ticked);

        let link = svg.append('g')
          .selectAll(".link")
          // .data(this.links)
          // .enter()
          .append("line")
          .attr("class", "link")
          // .attr("stroke", "#000")
          // .attr("stroke-width", 1.5)
          // .selectAll("line");

        let node = svg.append('g')
          .selectAll('.node')
          // .data(this.nodes)
          // .enter()
          .append('g')
          .attr("class", "node")
          // .attr("stroke", "#fff")
          // .attr("stroke-width", 1.5)
          // .selectAll("circle")
          
        function ticked() {
          link.attr("x1", d => d.source.x)
            .attr("y1", d => d.source.y)
            .attr("x2", d => d.target.x)
            .attr("y2", d => d.target.y)

          node.attr("transform", d => "translate(" + d.x + "," + d.y + ")")
        }

        // Terminate the force layout when this cell re-runs.
        // invalidation.then(() => simulation.stop());

        return Object.assign(svg.node(), {
          update({ nodes, links }) {
            // Make a shallow copy to protect against mutation, while
            // recycling old nodes to preserve position and velocity.
            const old = new Map(node.data().map(d => [d.id, d]));

            nodes = nodes.map(d => Object.assign(old.get(d.id) || {}, d));
            links = links.map(d => Object.assign({}, d));

            node = node
              .data(nodes, d => d.id)
              .join(enter => {
                const group = enter.append('g')
                  .attr('class', 'node')

                group.append("circle")
                    .attr("r", 8)
                    .attr("fill", d => color(d.id))

                group.append("text")
                    .attr("dx", d => (d.id.length / 2) * -10)
                    .attr("dy", 25)
                    .text(d => d.id)

                return group
              })

            link = link
              .data(links, d => [d.source, d.target])
              .join(enter => enter
                .append('line')
                .attr('class', 'link')
              )

            simulation.nodes(nodes);
            simulation.force("link").links(links);
            simulation.alpha(1).restart();
          }
        });
      })()

      return chart
    },
  },
  methods: {
    update() {
      this.chart.update({
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
    this.update()
  }
}
</script>

<style>
svg {
  border: 1px solid black;
}

.link {
  stroke: #888888;
}

.node text {
  pointer-events: none;
  font-size: 20px;
  color: black;
}
</style>
