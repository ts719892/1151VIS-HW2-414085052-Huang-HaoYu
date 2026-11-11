<script setup>
import {
  onMounted,
  ref,
  watch,
} from 'vue'

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

const chartRef =
  ref(null)

const containerRef =
  ref(null)

const width = 900
const height = 420

const margin = {
  top: 55,
  right: 45,
  bottom: 55,
  left: 95,
}

let initialized =
  false

onMounted(() => {
  initializeChart()
  updateChart()
})

watch(
  () => props.data,
  () => {
    updateChart()
  },
  {
    deep: true,
  }
)

function initializeChart() {
  if (
    !chartRef.value ||
    initialized
  ) {
    return
  }

  const svg = d3
    .select(chartRef.value)
    .attr(
      'viewBox',
      `0 0 ${width} ${height}`
    )

  svg
    .append('g')
    .attr(
      'class',
      'grid'
    )

  svg
    .append('g')
    .attr(
      'class',
      'x-axis'
    )

  svg
    .append('g')
    .attr(
      'class',
      'y-axis'
    )

  svg
    .append('path')
    .attr(
      'class',
      'line'
    )

  initialized = true
}

function updateChart() {
  if (
    !chartRef.value ||
    props.data.length === 0
  ) {
    return
  }

  const data =
    props.data

  const svg = d3
    .select(
      chartRef.value
    )

  const x = d3
    .scalePoint()
    .domain(
      data.map(
        (d) => d.month
      )
    )
    .range([
      margin.left,
      width -
        margin.right,
    ])

  const maxValue =
    d3.max(
      data,
      (d) =>
        d.visitors
    ) || 0

  const y = d3
    .scaleLinear()
    .domain([
      0,
      maxValue,
    ])
    .nice()
    .range([
      height -
        margin.bottom,

      margin.top,
    ])

  const transition =
    svg
      .transition()
      .duration(800)
      .ease(
        d3.easeCubicOut
      )

  // ==========================
  // Grid
  // ==========================

  svg
    .select('.grid')
    .attr(
      'transform',
      `translate(${margin.left},0)`
    )
    .transition(
      transition
    )
    .call(
      d3
        .axisLeft(y)
        .ticks(5)
        .tickSize(
          -(
            width -
            margin.left -
            margin.right
          )
        )
        .tickFormat('')
    )

  // ==========================
  // X Axis
  // ==========================

  svg
    .select('.x-axis')
    .attr(
      'transform',
      `translate(0,${
        height -
        margin.bottom
      })`
    )
    .transition(
      transition
    )
    .call(
      d3
        .axisBottom(x)
        .tickFormat(
          (d) =>
            `${d}月`
        )
    )

  // ==========================
  // Y Axis
  // ==========================

  svg
    .select('.y-axis')
    .attr(
      'transform',
      `translate(${margin.left},0)`
    )
    .transition(
      transition
    )
    .call(
      d3
        .axisLeft(y)
        .ticks(5)
        .tickFormat(
          formatWan
        )
    )

  // ==========================
  // Line
  // ==========================

  const line = d3
    .line()
    .x(
      (d) =>
        x(d.month)
    )
    .y(
      (d) =>
        y(d.visitors)
    )
    .curve(
      d3.curveMonotoneX
    )

  svg
    .select('.line')
    .datum(data)
    .attr(
      'fill',
      'none'
    )
    .attr(
      'stroke',
      '#2563eb'
    )
    .attr(
      'stroke-width',
      3
    )
    .attr(
      'stroke-linejoin',
      'round'
    )
    .attr(
      'stroke-linecap',
      'round'
    )
    .transition(
      transition
    )
    .attr(
      'd',
      line
    )

  const maxVisitors =
    d3.max(
      data,
      (d) =>
        d.visitors
    )

  const minVisitors =
    d3.min(
      data,
      (d) =>
        d.visitors
    )

  // ==========================
  // Points
  // ==========================

  const points = svg
    .selectAll(
      '.point'
    )
    .data(
      data,
      (d) => d.month
    )

  points
    .join(
      (enter) =>
        enter
          .append(
            'circle'
          )
          .attr(
            'class',
            'point'
          )
          .attr(
            'cx',
            (d) =>
              x(d.month)
          )
          .attr(
            'cy',
            height -
              margin.bottom
          )
          .attr(
            'r',
            0
          )
          .attr(
            'stroke',
            '#ffffff'
          )
          .attr(
            'stroke-width',
            2
          )
          .call(
            (enter) =>
              enter
                .transition(
                  transition
                )
                .attr(
                  'cy',
                  (d) =>
                    y(
                      d.visitors
                    )
                )
                .attr(
                  'r',
                  6
                )
          ),

      (update) =>
        update.call(
          (update) =>
            update
              .transition(
                transition
              )
              .attr(
                'cx',
                (d) =>
                  x(
                    d.month
                  )
              )
              .attr(
                'cy',
                (d) =>
                  y(
                    d.visitors
                  )
              )
              .attr(
                'r',
                6
              )
        ),

      (exit) =>
        exit.remove()
    )
    .attr(
      'fill',
      (d) => {
        if (
          d.visitors ===
          maxVisitors
        ) {
          return '#ef4444'
        }

        if (
          d.visitors ===
          minVisitors
        ) {
          return '#16a34a'
        }

        return '#2563eb'
      }
    )
    .on(
      'mouseover',
      function (
        event,
        d
      ) {
        d3
          .select(this)
          .transition()
          .duration(150)
          .attr(
            'r',
            9
          )

        showTooltip(
          event,
          d
        )
      }
    )
    .on(
      'mousemove',
      function (
        event,
        d
      ) {
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
          .transition()
          .duration(150)
          .attr(
            'r',
            6
          )

        hideTooltip()
      }
    )

  // ==========================
  // Labels
  // ==========================

  const labels = svg
    .selectAll(
      '.point-label'
    )
    .data(
      data,
      (d) => d.month
    )

  labels
    .join(
      (enter) =>
        enter
          .append(
            'text'
          )
          .attr(
            'class',
            'point-label'
          )
          .attr(
            'x',
            (d) =>
              x(d.month)
          )
          .attr(
            'y',
            height -
              margin.bottom
          )
          .attr(
            'text-anchor',
            'middle'
          )
          .attr(
            'font-size',
            12
          )
          .attr(
            'fill',
            '#334155'
          )
          .text(
            (d) =>
              formatWan(
                d.visitors
              )
          )
          .call(
            (enter) =>
              enter
                .transition(
                  transition
                )
                .attr(
                  'y',
                  (d) =>
                    y(
                      d.visitors
                    ) -
                    16
                )
          ),

      (update) =>
        update
          .text(
            (d) =>
              formatWan(
                d.visitors
              )
          )
          .call(
            (update) =>
              update
                .transition(
                  transition
                )
                .attr(
                  'x',
                  (d) =>
                    x(
                      d.month
                    )
                )
                .attr(
                  'y',
                  (d) =>
                    y(
                      d.visitors
                    ) -
                    16
                )
          ),

      (exit) =>
        exit.remove()
    )
}

function formatWan(
  value
) {
  return (
    Math.round(
      value / 10000
    ).toLocaleString(
      'zh-TW'
    ) + '萬'
  )
}

function showTooltip(
  event,
  d
) {
  if (
    !containerRef.value
  ) {
    return
  }

  const tooltip = d3
    .select(
      containerRef.value
    )
    .select(
      '.tooltip'
    )

  const [
    mouseX,
    mouseY,
  ] = d3.pointer(
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
      `${
        mouseX + 15
      }px`
    )
    .style(
      'top',
      `${
        mouseY - 10
      }px`
    )
    .html(`
      <strong>
        ${props.year} 年 ${d.month} 月
      </strong>

      <br>

      遊客人次：
      ${d3.format(',')(d.visitors)}
    `)
}

function hideTooltip() {
  if (
    !containerRef.value
  ) {
    return
  }

  d3
    .select(
      containerRef.value
    )
    .select(
      '.tooltip'
    )
    .style(
      'opacity',
      0
    )
}
</script>

<template>
  <section>
    <h2>
      {{ year }} 年每月遊客人次趨勢
    </h2>

    <p class="description">
      統計全台主要觀光遊憩據點每月遊客人次，
      切換年份可觀察不同年度的旅遊趨勢。
    </p>

    <div class="legend">
      <span>
        🔵 一般月份
      </span>

      <span>
        🔴 最高月份
      </span>

      <span>
        🟢 最低月份
      </span>
    </div>

    <div
      ref="containerRef"
      class="chart-container"
    >
      <svg
        ref="chartRef"
      ></svg>

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

.legend {
  display: flex;

  flex-wrap: wrap;

  gap: 20px;

  margin: 20px 0;

  font-size: 13px;
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

:deep(.grid line) {
  stroke: #e2e8f0;
}

:deep(.grid .domain) {
  stroke: none;
}
</style>