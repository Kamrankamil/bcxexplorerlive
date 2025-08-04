<script lang="ts" setup>
import ApexCharts from 'vue3-apexcharts';
import { computed, ref, onMounted } from 'vue';
import { useBaseStore } from '@/stores';

const baseStore = useBaseStore();
const blockData = ref<{ height: number; txs: number }[]>([]);

onMounted(async () => {
  const historicalBlocks = await fetchHistoricalBlocks();
  const recentBlocks = baseStore.recents.map(x => ({
    height: x.block.header.height,
    txs: x.block.data?.txs?.length || 0,
  }));

  const historical = historicalBlocks
    .filter(b => b.num_txs > 0)
    .map(b => ({
      height: b.height,
      txs: b.num_txs,
    }));

  blockData.value = [...recentBlocks, ...historical]
    .sort((a, b) => b.height - a.height)
    .slice(0, 50);
});

async function fetchHistoricalBlocks(): Promise<{ height: number; num_txs: number }[]> {
  try {
    const res = await fetch('http://65.109.78.203:8080/v1/graphql', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        query: `
          query {
            block(where: { num_txs: { _gt: 0 } }, order_by: { height: desc }, limit: 1000) {
              height
              num_txs
            }
          }
        `,
      }),
    });

    const json = await res.json();
    return json?.data?.block || [];
  } catch (err) {
    console.error('Failed to fetch historical blocks:', err);
    return [];
  }
}

const options = computed(() => ({
  chart: {
    type: 'bar',
    height: 250,
    toolbar: { show: false },
  },
  plotOptions: {
    bar: {
      borderRadius: 6,
      columnWidth: '55%',
      distributed: true,
      dataLabels: {
        position: 'center',
      },
    },
  },
  dataLabels: {
    enabled: true,
    formatter: (val: number) => `${val} tx`,
    style: {
      fontSize: '12px',
      colors: ['#ffffff'], // white text inside bars
      fontWeight: 'bold',
    },
  },
  colors: ['#6366f1', '#7c3aed', '#3b82f6'],
  xaxis: {
    categories: blockData.value.map(b => `#${b.height}`),
    labels: {
      show: true,
      rotate: -15,
      style: {
        fontSize: '11px',
        colors: '#64748B',
      },
    },
    axisTicks: { show: false },
    axisBorder: { show: false },
  },
  yaxis: {
    title: { text: 'Transactions' },
    labels: {
      style: { colors: ['#64748B'], fontSize: '12px' },
    },
  },
  grid: {
    strokeDashArray: 4,
    padding: {
      top: 10,
      bottom: 10,
      left: 12,
      right: 12,
    },
  },
  tooltip: {
    y: {
      formatter: (val: number) => `${val} transactions`,
    },
  },
}));


const series = computed(() => [{
  name: 'Transactions',
  data: blockData.value.map(b => b.txs),
}]);
</script>

<template>
  <div class="bg-white dark:bg-gray-900 rounded-xl shadow-md p-4">
    <h2 class="text-base font-semibold text-gray-700 dark:text-gray-200 mb-2">
      Block Transactions (Last 50)
    </h2>
    <ApexCharts type="bar" height="250" :options="options" :series="series" />
  </div>
</template>
