<script setup>
import { onMounted, ref, watch } from 'vue'
import * as d3 from 'd3'

import BarChart from './components/BarChart.vue'
import LineChart from './components/LineChart.vue'

const rawData = ref([])

const cityData = ref([])
const monthlyData = ref([])

const years = ref([])
const selectedYear = ref(null)

const loading = ref(true)
const error = ref('')

function parseNumber(value) {
  return Number(String(value ?? 0).replaceAll(',', '')) || 0
}

function updateData() {
  if (!selectedYear.value) return

  const yearData = rawData.value.filter(
    (d) => Number(d['年別']) === Number(selectedYear.value)
  )

  // ==========================
  // 各縣市旅遊人次
  // ==========================
  const grouped = d3.rollups(
    yearData.filter((d) => d['縣市']),
    (rows) => d3.sum(rows, (d) => parseNumber(d['合計'])),
    (d) => d['縣市'].split('\n')[0].trim()
  )

  cityData.value = grouped
    .map(([city, visitors]) => ({
      city,
      visitors,
    }))
    .sort((a, b) => b.visitors - a.visitors)

  // ==========================
  // 1 ~ 12 月旅遊人次
  // ==========================
  monthlyData.value = d3.range(1, 13).map((month) => ({
    month,
    visitors: d3.sum(
      yearData,
      (d) => parseNumber(d[`${month}月`])
    ),
  }))
}

onMounted(async () => {
  try {
    rawData.value = await d3.csv('/tourism.csv')

    years.value = [
      ...new Set(
        rawData.value
          .map((d) => Number(d['年別']))
          .filter((year) => !Number.isNaN(year))
      ),
    ].sort((a, b) => b - a)

    if (years.value.length > 0) {
      selectedYear.value = years.value[0]
      updateData()
    }
  } catch (e) {
    console.error(e)
    error.value = e.message
  } finally {
    loading.value = false
  }
})

watch(selectedYear, () => {
  updateData()
})
</script>

<template>
  <main>
    <header>
      <h1>{{ selectedYear }} 台灣觀光旅遊人次分析</h1>

      <p class="subtitle">
        資料來源：交通部觀光署
      </p>

      <div class="year-selector">
        <label for="year">
          選擇年份：
        </label>

        <select
          id="year"
          v-model.number="selectedYear"
        >
          <option
            v-for="year in years"
            :key="year"
            :value="year"
          >
            {{ year }}
          </option>
        </select>
      </div>
    </header>

    <p v-if="loading">
      正在載入資料...
    </p>

    <p v-else-if="error" class="error">
      資料讀取失敗：{{ error }}
    </p>

    <template v-else>
      <BarChart
        :data="cityData"
        :year="selectedYear"
      />

      <LineChart
        :data="monthlyData"
        :year="selectedYear"
      />
    </template>
  </main>
</template>

<style>
body {
  margin: 0;
  background: #f3f6fa;
  color: #1e293b;
  font-family:
    Arial,
    "Microsoft JhengHei",
    sans-serif;
}

#app {
  width: 100%;
}

main {
  max-width: 1150px;
  margin: 0 auto;
  padding: 30px 20px;
}

header {
  margin-bottom: 30px;
}

h1 {
  margin-bottom: 8px;
}

.subtitle {
  color: #64748b;
}

.year-selector {
  margin-top: 20px;
}

.year-selector label {
  font-weight: bold;
  margin-right: 10px;
}

.year-selector select {
  padding: 8px 14px;
  border: 1px solid #cbd5e1;
  border-radius: 6px;
  background: white;
  font-size: 16px;
}

section {
  margin-bottom: 28px;
  padding: 28px;
  background: white;
  border-radius: 12px;
}

.error {
  color: #dc2626;
}
</style>