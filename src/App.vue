<script setup>
import { onMounted, ref } from 'vue';
import * as d3 from 'd3';
import BarChart from './components/BarChart.vue';
import LineChart from './components/LineChart.vue';

const cityData = ref([]);
const monthlyData = ref([]);
const loading = ref(true);
const error = ref('');

onMounted(async () => {
  try {
    const rawData = await d3.csv('/tourism.csv');

    const data2024 = rawData.filter((d) => +d['年別'] === 2024);

    const grouped = d3.rollups(
      data2024.filter((d) => d['縣市']),
      (rows) => d3.sum(rows, (d) => +d['合計'] || 0),
      (d) => d['縣市'].split('\n')[0].trim()
    );

    cityData.value = grouped
      .map(([city, visitors]) => ({
        city,
        visitors,
      }))
      .sort((a, b) => b.visitors - a.visitors);
    // 統計 2024 年每月全台遊客人次
    monthlyData.value = d3.range(1, 13).map((month) => ({
      month,
      visitors: d3.sum(
        data2024,
        (d) => Number(String(d[`${month}月`] ?? 0).replaceAll(',', '')) || 0
      ),
    }));

    console.table(monthlyData.value);
  } catch (e) {
    error.value = e.message;
  } finally {
    loading.value = false;
  }
});
</script>

<template>
  <main>
    <header>
      <h1>2024 台灣觀光旅遊人次分析</h1>
      <p>資料來源：交通部觀光署</p>
    </header>

    <p v-if="loading">正在載入資料...</p>
    <p v-else-if="error">{{ error }}</p>

    <BarChart v-else :data="cityData" />
    <LineChart :data="monthlyData" />
  </main>
</template>

<style>
body {
  background: #f3f6fa;
  color: #1e293b;
  margin: 0;
  font-family: Arial, sans-serif;
}

#app {
  max-width: none;
  margin: 0;
  padding: 0;
  text-align: left;
}

main {
  max-width: 1150px;
  margin: auto;
  padding: 30px 20px;
}

section {
  background: white;
  border-radius: 12px;
  padding: 28px;
  margin-bottom: 25px;
}

svg {
  width: 100%;
  height: auto;
  display: block;
}
</style>
