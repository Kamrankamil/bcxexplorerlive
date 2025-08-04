<script lang="ts" setup>
import ApexCharts from 'vue3-apexcharts';
import { ref, computed, onMounted } from 'vue';

interface Block {
  timestamp: string;
  num_txs: number;
}

interface ChartPoint {
  date: string;
  value: number;
}

const chartData = ref<ChartPoint[]>([]);

onMounted(async () => {
  const res = await fetch('http://65.109.78.203:8080/api/rest/block');
  const json = await res.json();
  const blocks: Block[] = json.block;

  // Group by day
  const dailyTotals: Record<string, number> = {};

  for (const block of blocks) {
    if (!block.timestamp || !block.num_txs) continue;
    const day = block.timestamp.slice(0, 10); // 'YYYY-MM-DD'
    dailyTotals[day] = (dailyTotals[day] || 0) + block.num_txs;
  }

  // Sort and map to chart points
  chartData.value = Object.entries(dailyTotals)
    .sort(([a], [b]) => a.localeCompare(b))
    .slice(-14)
    .map(([date, value]) => ({
      date: new Date(date).toLocaleDateString('en-US', { month: 'short', day: 'numeric' }),
      value
    }));
});

const options = computed(() => ({
  chart: {
    type: 'line',
    toolbar: { show: false },
    zoom: { enabled: false }
  },
  stroke: {
    curve: 'smooth',
    width: 2
  },
  markers: {
    size: 0
  },
  tooltip: {
    y: {
      formatter: (val: number) => val.toLocaleString()
    }
  },
  xaxis: {
    categories: chartData.value.map(d => d.date),
    title: {
      text: 'Date',
      style: { color: '#64748B' }
    },
    labels: {
      style: { fontSize: '12px', colors: '#64748B' }
    }
  },
  yaxis: {
    title: {
      text: 'Transactions',
      style: { color: '#64748B' }
    },
    labels: {
      formatter: (val: number) => `${Math.round(val / 1000)}k`,
      style: { fontSize: '12px', colors: '#64748B' }
    }
  },
  grid: {
    show: true,
    borderColor: '#E5E7EB',
    strokeDashArray: 4
  },
  colors: ['#4F46E5']
}));

const series = computed(() => [{
  name: 'Transactions',
  data: chartData.value.map(d => d.value)
}]);
</script>

<template>
  <div class="bg-white dark:bg-gray-900 rounded-xl shadow-md p-4">
    <h2 class="text-sm font-medium text-gray-500 dark:text-gray-300 mb-2">
      TRANSACTION HISTORY IN 14 DAYS
    </h2>
    <ApexCharts type="line" height="200" :options="options" :series="series" />
  </div>
</template>
