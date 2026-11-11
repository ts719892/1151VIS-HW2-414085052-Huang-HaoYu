<script setup>
import { onMounted, ref, watch } from 'vue';
import * as d3 from 'd3';

// 接收 App.vue 傳入的月份資料
const props = defineProps({
  data: {
    type: Array,
    required: true,
  },
});

// 使用 ref 定位 SVG，避免與其他圖表互相影響
const chartRef = ref(null);

onMounted(() => {
  drawChart();
});

// 未來如果資料變更，可以自動重新繪製
watch(() => props.data, drawChart);

function drawChart() {
  if (!chartRef.value || !props.data.length) return;

  const data = props.data;
  const width = 900;
  const height = 420;
  const margin = {
    top: 50,
    right: 45,
    bottom: 55,
    left: 95,
  };

  const svg = d3
    .select(chartRef.value)
    .attr('viewBox', `0 0 ${width} ${height}`);

  // 避免重複產生圖表
  svg.selectAll('*').remove();

  // X 軸：1～12 月
  const x = d3
    .scalePoint()
    .domain(data.map((d) => d.month))
    .range([margin.left, width - margin.right]);

  // Y 軸：遊客人次
  const y = d3
    .scaleLinear()
    .domain([0, d3.max(data, (d) => d.visitors)])
    .nice()
    .range([height - margin.bottom, margin.top]);

  const formatWan = (value) =>
    Math.round(value / 10000).toLocaleString('zh-TW') + '萬';

  // 水平網格線
  svg
    .append('g')
    .attr('class', 'grid')
    .attr('transform', `translate(${margin.left},0)`)
    .call(
      d3
        .axisLeft(y)
        .ticks(5)
        .tickSize(-(width - margin.left - margin.right))
        .tickFormat('')
    );

  // X 軸
  svg
    .append('g')
    .attr('transform', `translate(0,${height - margin.bottom})`)
    .call(d3.axisBottom(x).tickFormat((d) => `${d}月`));

  // Y 軸
  svg
    .append('g')
    .attr('transform', `translate(${margin.left},0)`)
    .call(d3.axisLeft(y).ticks(5).tickFormat(formatWan));

  // 找出最高和最低月份
  const maxVisitors = d3.max(data, (d) => d.visitors);
  const minVisitors = d3.min(data, (d) => d.visitors);

  // 建立折線路徑
  const line = d3
    .line()
    .x((d) => x(d.month))
    .y((d) => y(d.visitors));

  svg
    .append('path')
    .datum(data)
    .attr('fill', 'none')
    .attr('stroke', '#2563eb')
    .attr('stroke-width', 3)
    .attr('stroke-linejoin', 'round')
    .attr('d', line);

  // 各月份資料點
  svg
    .selectAll('.point')
    .data(data)
    .join('circle')
    .attr('class', 'point')
    .attr('cx', (d) => x(d.month))
    .attr('cy', (d) => y(d.visitors))
    .attr('r', 6)
    .attr('fill', (d) =>
      d.visitors === maxVisitors
        ? '#ef4444'
        : d.visitors === minVisitors
        ? '#16a34a'
        : '#2563eb'
    )
    .attr('stroke', '#ffffff')
    .attr('stroke-width', 2);

  // 每個點顯示實際人次
  svg
    .selectAll('.point-label')
    .data(data)
    .join('text')
    .attr('class', 'point-label')
    .attr('x', (d) => x(d.month))
    .attr('y', (d) => y(d.visitors) - 16)
    .attr('text-anchor', 'middle')
    .attr('font-size', 12)
    .attr('fill', '#334155')
    .text((d) => formatWan(d.visitors));
}
</script>

<template>
  <section>
    <h2>2024 年每月遊客人次趨勢</h2>

    <p class="description">
      統計全台主要觀光遊憩據點每月遊客人次， 觀察不同月份的旅遊變化。
    </p>

    <div class="legend">
      <span>🔵 一般月份</span>
      <span>🔴 最高月份</span>
      <span>🟢 最低月份</span>
    </div>

    <svg ref="chartRef"></svg>

    <p class="source">資料來源：交通部觀光署｜單位：萬人次</p>
  </section>
</template>

<style scoped>
.description {
  color: #64748b;
  line-height: 1.8;
}

.legend {
  display: flex;
  gap: 20px;
  flex-wrap: wrap;
  font-size: 13px;
  margin: 20px 0;
}

svg {
  display: block;
  width: 100%;
  height: auto;
}

.source {
  text-align: right;
  font-size: 12px;
  color: #94a3b8;
}

:deep(.grid line) {
  stroke: #e2e8f0;
}

:deep(.grid .domain) {
  stroke: none;
}
</style>
