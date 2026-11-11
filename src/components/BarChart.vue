<script setup>
import { onMounted, ref, watch } from 'vue'
import * as d3 from 'd3'

const props = defineProps({
  data: {
    type: Array,
    required: true,
  },

  year: {
    type: Number,
    required: true,
  },
})

const chartRef = ref(null)
const containerRef = ref(null)

const width = 900

onMounted(() => {
  drawChart()
})

watch(
  () => props.data,
  () => {
    drawChart()
  },
  {
    deep: true,
  }
)

function drawChart() {
  if (!chartRef.value || props.data.length === 0) return

  const data = props.data

  const rowHeight = 34

  const margin = {
    top: 20,
    right: 110,
    bottom: 60,
    left: 100,
  }

  const height =
    data.length * rowHeight +
    margin.top +
    margin.bottom

  const svg = d3
    .select(chartRef.value)
    .attr('viewBox', `0 0 ${width} ${height}`)

  const x = d3
    .scaleLinear()
    .domain([
      0,
      d3.max(data, (d) => d.visitors) || 0,
    ])
    .nice()
    .range([
      margin.left,
      width - margin.right,
    ])

  const y = d3
    .scaleBand()
    .domain(
      data.map((d) => d.city)
    )
    .range([
      margin.top,
      height - margin.bottom,
    ])
    .padding(0.2)

  const transition = svg
    .transition()
    .duration(800)
    .ease(d3.easeCubicOut)

  // ==========================
  // X Axis
  // ==========================

  let xAxis = svg.select('.x-axis')

  if (xAxis.empty()) {
    xAxis = svg
      .append('g')
      .attr('class', 'x-axis')
  }

  xAxis
    .attr(
      'transform',
      `translate(0,${height - margin.bottom})`
    )
    .transition(transition)
    .call(
      d3
        .axisBottom(x)
        .ticks(6)
        .tickFormat((d) => {
          return `${Math.round(d / 10000).toLocaleString()}萬`
        })
    )

  // ==========================
  // Y Axis
  // ==========================

  let yAxis = svg.select('.y-axis')

  if (yAxis.empty()) {
    yAxis = svg
      .append('g')
      .attr('class', 'y-axis')
  }

  yAxis
    .attr(
      'transform',
      `translate(${margin.left},0)`
    )
    .transition(transition)
    .call(d3.axisLeft(y))

  // ==========================
  // Bar
  // ==========================

  const bars = svg
    .selectAll('.bar')
    .data(
      data,
      (d) => d.city
    )

  bars
    .join(
      (enter) =>
        enter
          .append('rect')
          .attr('class', 'bar')
          .attr('x', margin.left)
          .attr(
            'y',
            (d) => y(d.city)
          )
          .attr(
            'height',
            y.bandwidth()
          )
          .attr('width', 0)
          .attr('rx', 3)
          .attr(
            'fill',
            '#14b8a6'
          )
          .call((enter) =>
            enter
              .transition(transition)
              .attr(
                'width',
                (d) =>
                  Math.max(
                    0,
                    x(d.visitors) -
                      margin.left
                  )
              )
          ),

      (update) =>
        update.call((update) =>
          update
            .transition(transition)
            .attr(
              'y',
              (d) => y(d.city)
            )
            .attr(
              'height',
              y.bandwidth()
            )
            .attr(
              'width',
              (d) =>
                Math.max(
                  0,
                  x(d.visitors) -
                    margin.left
                )
            )
        ),

      (exit) =>
        exit.call((exit) =>
          exit
            .transition()
            .duration(400)
            .attr('width', 0)
            .remove()
        )
    )
    .on(
      'mouseover',
      function (event, d) {
        d3
          .select(this)
          .attr(
            'fill',
            '#0f766e'
          )

        showTooltip(
          event,
          d
        )
      }
    )
    .on(
      'mousemove',
      function (event, d) {
        showTooltip(
          event,
          d
        )
      }
    )
    .on(
      'mouseout',
      function () {
        d3
          .select(this)
          .attr(
            'fill',
            '#14b8a6'
          )

        hideTooltip()
      }
    )

  // ==========================
  // 數值標籤
  // ==========================

  const labels = svg
    .selectAll('.value-label')
    .data(
      data,
      (d) => d.city
    )

  labels
    .join(
      (enter) =>
        enter
          .append('text')
          .attr(
            'class',
            'value-label'
          )
          .attr(
            'x',
            margin.left
          )
          .attr(
            'y',
            (d) =>
              y(d.city) +
              y.bandwidth() / 2
          )
          .attr(
            'dominant-baseline',
            'middle'
          )
          .attr(
            'font-size',
            11
          )
          .attr(
            'fill',
            '#334155'
          )
          .text((d) =>
            formatWan(
              d.visitors
            )
          )
          .call((enter) =>
            enter
              .transition(transition)
              .attr(
                'x',
                (d) =>
                  x(d.visitors) + 7
              )
          ),

      (update) =>
        update
          .text((d) =>
            formatWan(
              d.visitors
            )
          )
          .call((update) =>
            update
              .transition(transition)
              .attr(
                'x',
                (d) =>
                  x(d.visitors) + 7
              )
              .attr(
                'y',
                (d) =>
                  y(d.city) +
                  y.bandwidth() / 2
              )
          ),

      (exit) => exit.remove()
    )
}

function formatWan(value) {
  return (
    Math.round(
      value / 10000
    ).toLocaleString('zh-TW') +
    '萬'
  )
}

function showTooltip(event, d) {
  if (!containerRef.value) return

  const tooltip = d3
    .select(
      containerRef.value
    )
    .select('.tooltip')

  const [x, y] = d3.pointer(
    event,
    containerRef.value
  )

  tooltip
    .style(
      'opacity',
      1
    )
    .style(
      'left',
      `${x + 15}px`
    )
    .style(
      'top',
      `${y - 10}px`
    )
    .html(`
      <strong>
        ${props.year} 年 ${d.city}
      </strong>

      <br>

      遊客人次：
      ${d3.format(',')(d.visitors)}
    `)
}

function hideTooltip() {
  if (!containerRef.value) return

  d3
    .select(
      containerRef.value
    )
    .select('.tooltip')
    .style(
      'opacity',
      0
    )
}
</script>

<template>
  <section>
    <h2>
      {{ year }} 年各縣市遊客人次排行榜
    </h2>

    <p class="description">
      統計各縣市主要觀光遊憩據點遊客人次，
      切換年份可觀察排名與人次變化。
    </p>

    <div
      ref="containerRef"
      class="chart-container"
    >
      <svg ref="chartRef"></svg>

      <div
        class="tooltip"
      ></div>
    </div>

    <p class="source">
      資料來源：交通部觀光署｜
      單位：人次
    </p>
  </section>
</template>

<style scoped>
.description {
  color: #64748b;
  line-height: 1.7;
}

.chart-container {
  position: relative;
}

svg {
  display: block;
  width: 100%;
  height: auto;
}

.tooltip {
  position: absolute;
  opacity: 0;
  pointer-events: none;

  padding: 10px 12px;

  background: rgba(
    15,
    23,
    42,
    0.94
  );

  color: white;

  border-radius: 8px;

  font-size: 14px;

  line-height: 1.6;

  white-space: nowrap;

  transition:
    opacity 0.15s;
}

.source {
  margin-top: 12px;

  text-align: right;

  color: #94a3b8;

  font-size: 12px;
}
</style>