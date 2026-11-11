
<script setup>
import { onMounted } from 'vue'
import * as d3 from 'd3'

const props = defineProps({
  data: {
    type: Array,
    required: true
  }
})

onMounted(() => {
  drawChart(props.data)
})

function drawChart(data) {
  const width = 900
  const height = 850
  const margin = {
    top: 20, right: 90, bottom: 60, left: 90
  }

  const svg = d3.select('#barChart')
    .attr('viewBox', `0 0 ${width} ${height}`)

  svg.selectAll('*').remove()

  const x = d3.scaleLinear()
    .domain([0, d3.max(data, d => d.visitors)])
    .nice()
    .range([margin.left, width - margin.right])

  const y = d3.scaleBand()
    .domain(data.map(d => d.city))
    .range([margin.top, height - margin.bottom])
    .padding(0.2)

  svg.selectAll('.bar')
    .data(data)
    .join('rect')
    .attr('class', 'bar')
    .attr('x', margin.left)
    .attr('y', d => y(d.city))
    .attr('width', d => x(d.visitors) - margin.left)
    .attr('height', y.bandwidth())
    .attr('fill', '#14b8a6')
    .attr('rx', 3)

  svg.append('g')
    .attr('transform', `translate(0,${height - margin.bottom})`)
    .call(d3.axisBottom(x).ticks(6).tickFormat(d3.format('.2s')))

  svg.append('g')
    .attr('transform', `translate(${margin.left},0)`)
    .call(d3.axisLeft(y))

  svg.selectAll('.value')
    .data(data)
    .join('text')
    .attr('class', 'value')
    .attr('x', d => x(d.visitors) + 6)
    .attr('y', d => y(d.city) + y.bandwidth() / 2)
    .attr('dominant-baseline', 'middle')
    .attr('font-size', 11)
    .text(d => (d.visitors / 10000).toFixed(0) + '萬')
}
</script>

<template>
  <section>
    <h2>各縣市遊客人次排行榜</h2>
    <p>
      2024 年全台 22 縣市主要觀光遊憩據點
      遊客人次排名。
    </p>
    <svg id="barChart"></svg>
  </section>
</template>
