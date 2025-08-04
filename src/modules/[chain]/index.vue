<script lang="ts" setup>
import { use } from 'echarts/core';
import { CanvasRenderer } from 'echarts/renderers';
import { BarChart, LineChart } from 'echarts/charts';
import { GridComponent, TooltipComponent, LegendComponent } from 'echarts/components';
import VChart from 'vue-echarts';
import TxsInBlocksChart from '@/components/charts/TxsInBlocksChart.vue';
use([CanvasRenderer, BarChart, LineChart, GridComponent, TooltipComponent, LegendComponent]);

import { ref } from 'vue';

// Chart 1: Daily User Transactions
const dailyUserTransactionsOption = ref({
  tooltip: { trigger: 'axis' },
  xAxis: {
    type: 'category',
    data: ['Jul 12', 'Jul 13', 'Jul 14', 'Jul 15', 'Jul 16', 'Jul 17', 'Jul 18', 'Jul 19', 'Jul 20', 'Jul 21'],
  },
  yAxis: {
    type: 'value',
    axisLabel: {
      formatter: (value: number) => {
        return value >= 1000000 ? value / 1000000 + 'M' : value;
      },
    },
  },
  series: [
    {
      data: [3800000, 4100000, 4300000, 4200000, 4000000, 4100000, 4400000, 4200000, 4400000, 4400000],
      type: 'bar',
      itemStyle: { color: '#6366f1' },
      barWidth: '50%',
    },
  ],
});

// Chart 2: Network Fees (APT)
const networkFeesOption = ref({
  tooltip: { trigger: 'axis' },
  legend: {
    data: ['Fee (APT)', 'Avg. Fee (APT)'],
    top: 10,
  },
  xAxis: {
    type: 'category',
    data: ['Jul 13', 'Jul 14', 'Jul 15', 'Jul 16', 'Jul 17', 'Jul 18', 'Jul 19', 'Jul 20', 'Jul 21'],
  },
  yAxis: [
    {
      type: 'value',
      name: 'Fee (APT)',
    },
    {
      type: 'value',
      name: 'Avg. Fee (APT)',
      position: 'right',
      axisLabel: {
        formatter: '{value}',
      },
    },
  ],
  series: [
    {
      name: 'Fee (APT)',
      type: 'bar',
      barWidth: 20,
      itemStyle: { color: '#6366f1' },
      data: [400, 420, 500, 460, 700, 900, 800, 1000, 600],
    },
    {
      name: 'Avg. Fee (APT)',
      type: 'line',
      yAxisIndex: 1,
      itemStyle: { color: '#9333ea' },
      lineStyle: { width: 2 },
      symbol: 'circle',
      symbolSize: 6,
      data: [0.00008, 0.0001, 0.00012, 0.00015, 0.00017, 0.0002, 0.00018, 0.00021, 0.0001],
    },
  ],
});

import MdEditor from 'md-editor-v3';
import PriceMarketChart from '@/components/charts/PriceMarketChart.vue';


import { Icon } from '@iconify/vue';
import {
  useBlockchain,
  useFormatter,
  useTxDialog,
  useWalletStore,
  useStakingStore,
  useParamStore,
  useBaseStore,
} from '@/stores';

import { useIndexModule, colorMap, tickerUrl } from './indexStore';

import TxsInBlocksChart from '@/components/charts/TxsInBlocksChart.vue';
import CardStatisticsVertical from '@/components/CardStatisticsVertical.vue';
import ProposalListItem from '@/components/ProposalListItem.vue';
import ArrayObjectElement from '@/components/dynamic/ArrayObjectElement.vue'

const props = defineProps(['chain']);
const tab = ref('blocks');

const blockchain = useBlockchain();
const store = useIndexModule();
const walletStore = useWalletStore();
const format = useFormatter();
const dialog = useTxDialog();
const stakingStore = useStakingStore();
const paramStore = useParamStore()
const coinInfo = computed(() => {
  return store.coinInfo;
});

onMounted(() => {
  store.loadDashboard();
  walletStore.loadMyAsset();
  paramStore.handleAbciInfo()
  // if(!(coinInfo.value && coinInfo.value.name)) {
  // }
});
const ticker = computed(() => store.coinInfo.tickers[store.tickerIndex]);
const base = useBaseStore();

const list = computed(() => base.recents);
const currName = ref("")
blockchain.$subscribe((m, s) => {
  if (s.chainName !== currName.value) {
    currName.value = s.chainName
    store.loadDashboard();
    walletStore.loadMyAsset();
    paramStore.handleAbciInfo()
  }
});
function shortName(name: string, id: string) {
  return name.toLowerCase().startsWith('ibc/') ||
    name.toLowerCase().startsWith('0x')
    ? id
    : name;
}

const comLinks = computed(()=> {
  return [
  {
    name: 'Website',
    icon: 'mdi-web',
    href: store.homepage,
  },
  {
    name: 'Twitter',
    icon: 'mdi-twitter',
    href: store.twitter,
  },
  {
    name: 'Telegram',
    icon: 'mdi-telegram',
    href: store.telegram,
  },
  {
    name: 'Github',
    icon: 'mdi-github',
    href: store.github,
  },
];
})

// wallet box
const change = computed(() => {
  const token = walletStore.balanceOfStakingToken;
  return token ? format.priceChanges(token.denom) : 0;
});
const color = computed(() => {
  switch (true) {
    case change.value > 0:
      return 'text-green-600';
    case change.value === 0:
      return 'text-grey-500';
    case change.value < 0:
      return 'text-red-600';
  }
});

function updateState() {
  walletStore.loadMyAsset()
}

function trustColor(v: string) {
  return `text-${colorMap(v)}`
}

const quantity = ref(100)
const qty = computed({
  get: () => {
    return parseFloat(quantity.value.toFixed(6))
  },
  set: val => {
    quantity.value = val
  }
})
const amount = computed({
  get: () => {
    return quantity.value * ticker.value.converted_last.usd || 0
  },
  set: val => {
    quantity.value = val / ticker.value.converted_last.usd || 0
  }
})



const appVersionPatched = computed(() => {
 const versionArray = Array.isArray(paramStore.appVersion?.items)
  ? [...paramStore.appVersion.items]
  : [];




  const patched = versionArray?.map(item => {
    if (item.subtitle === 'name' && item.value === 'evmos') {

      return { ...item, value: 'bcx' }
    }
    return item
  })

 
  return patched
})


</script>

<template>
  <div>
    <div v-if="coinInfo && coinInfo.name" class="bg-base-100 rounded shadow">
      <div class="grid grid-cols-2 md:grid-cols-3 p-4">
        <div class="col-span-2 md:col-span-1">
          <div class="text-xl font-semibold text-main">
            {{ coinInfo.name }} (<span class="uppercase">{{
              coinInfo.symbol
            }}</span>)
          </div>
          <div class="text-xs mt-2">
            {{ $t('index.rank') }}:
            <div class="badge text-xs badge-error bg-[#fcebea] dark:bg-[#41384d] text-red-400">
              #{{ coinInfo.market_cap_rank }}
            </div>
          </div>

          <div class="my-4 flex flex-wrap items-center">
            <a v-for="(item, index) of comLinks" :key="index" :href="item.href"
              class="link link-primary px-2 py-1 rounded-sm no-underline hover:text-primary hover:bg-gray-100 dark:hover:bg-slate-800 flex items-center">
              <Icon :icon="item?.icon" />
              <span class="ml-1 text-sm uppercase">{{ item?.name }}</span>
            </a>
          </div>

          <div>
            <div class="dropdown dropdown-hover w-full">
              <label>
                <div
                  class="bg-gray-100 dark:bg-[#384059] flex items-center justify-between px-4 py-2 cursor-pointer rounded">
                  <div>
                    <div class="font-semibold text-xl text-[#666] dark:text-white">
                      {{ ticker?.market?.name || '' }}
                    </div>
                    <div class="text-info text-sm">
                      {{ shortName(ticker?.base, ticker?.coin_id) }}/{{
                        shortName(ticker?.target, ticker?.target_coin_id)
                      }}
                    </div>
                  </div>

                  <div class="text-right">
                    <div class="text-xl font-semibold text-[#666] dark:text-white">
                      ${{ ticker?.converted_last?.usd }}
                    </div>
                    <div class="text-sm" :class="store.priceColor">
                      {{ store.priceChange }}%
                    </div>
                  </div>
                </div>
              </label>
              <div class="dropdown-content pt-1">
                <div class="h-64 overflow-auto w-full shadow rounded">
                  <ul class="menu w-full bg-gray-100 rounded dark:bg-[#384059]">
                    <li v-for="(item, index) in store.coinInfo.tickers" :key="index" @click="store.selectTicker(index)">
                      <div class="flex items-center justify-between hover:bg-base-100">
                        <div class="flex-1">
                          <div class="text-main text-sm" :class="trustColor(item.trust_score)">
                            {{ item?.market?.name }}
                          </div>
                          <div class="text-sm text-gray-500 dark:text-gray-400">
                            {{ shortName(item?.base, item?.coin_id) }}/{{
                              shortName(item?.target, item?.target_coin_id)
                            }}
                          </div>
                        </div>

                        <div class="text-base text-main">
                           ${{ item?.converted_last?.usd }}
                        </div>
                      </div>
                    </li>
                  </ul>
                </div>
              </div>
            </div>

            <div class="flex">
              <label class="btn btn-primary !px-1 my-5 mr-2" for="calculator">
                <svg class="w-8 h-8" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="#ffffff" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><g id="SVGRepo_bgCarrier" stroke-width="0"></g><g id="SVGRepo_tracerCarrier" stroke-linecap="round" stroke-linejoin="round"></g><g id="SVGRepo_iconCarrier"> <rect x="4" y="2" width="16" height="20" rx="2"></rect> <line x1="8" x2="16" y1="6" y2="6"></line> <line x1="16" x2="16" y1="14" y2="18"></line> <path d="M16 10h.01"></path> <path d="M12 10h.01"></path> <path d="M8 10h.01"></path> <path d="M12 14h.01"></path> <path d="M8 14h.01"></path> <path d="M12 18h.01"></path> <path d="M8 18h.01"></path> </g></svg>
              </label>
              <!-- Put this part before </body> tag -->
              <input type="checkbox" id="calculator" class="modal-toggle" />
              <div class="modal">
                <div class="modal-box">
                  <h3 class="text-lg font-bold">{{ $t('index.price_calculator') }}</h3>
                  <div class="flex flex-col w-full mt-5">
                    <div class="grid h-20 flex-grow card rounded-box place-items-center">
                      <div class="join w-full">
                        <label class="join-item btn">
                          <span class="uppercase">{{ coinInfo.symbol }}</span>
                        </label>
                        <input type="number" v-model="qty" min="0" placeholder="Input a number" class="input grow input-bordered join-item" />
                      </div>
                    </div>
                    <div class="divider">=</div>
                    <div class="grid h-20 flex-grow card rounded-box place-items-center">
                      <div class="join w-full">
                        <label class="join-item btn">
                          <span>USD</span>
                        </label>
                        <input type="number" v-model="amount" min="0" placeholder="Input amount" class="join-item grow input input-bordered" />
                      </div>
                    </div>
                  </div>
                </div>
                <label class="modal-backdrop" for="calculator">{{ $t('index.close') }}</label>
              </div>
              <a class="my-5 !text-white btn grow" :class="{'!btn-success': store.trustColor === 'green', '!btn-warning': store.trustColor === 'yellow'}" :href="tickerUrl(ticker.trade_url)"
                target="_blank">
                {{ $t('index.buy') }} {{ coinInfo.symbol || '' }}
              </a>
            </div>
          </div>
        </div>

        <div class="col-span-2">
          <PriceMarketChart />
        </div>
      </div>
      <div class="h-[1px] w-full bg-gray-100 dark:bg-[#384059]"></div>
      <div class="max-h-[250px] overflow-auto p-4 text-sm">
        <MdEditor :model-value="coinInfo.description?.en" previewOnly></MdEditor>
      </div>
      <div class="mx-4 flex flex-wrap items-center">
        <div v-for="tag in coinInfo.categories"
          class="mr-2 mb-4 text-xs bg-gray-100 dark:bg-[#384059] px-3 rounded-full py-1">
          {{ tag }}
        </div>
      </div>
    </div>


   <!-- Statistic Cards -->
  <!-- Etherscan-style Horizontal Card -->
<!-- Etherscan-style Info Panel -->
<!-- Dashboard Overview Box -->
<div class="bg-base-100 rounded-xl shadow p-4 mt-6">
  <div class="grid grid-cols-1 lg:grid-cols-4 gap-6">
    <!-- Left: Stats Blocks (spanning 3 columns) -->
    <div class="lg:col-span-3 grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
      <CardStatisticsVertical v-bind="store.stats[0]" />
      <CardStatisticsVertical v-bind="store.stats[1]" />
      <CardStatisticsVertical v-bind="store.stats[2]" />
      <CardStatisticsVertical v-bind="store.stats[3]" />
      <CardStatisticsVertical v-bind="store.stats[4]" />
      <CardStatisticsVertical v-bind="store.stats[5]" />
    </div>

    <!-- Right: 14-day Line Chart -->
  <div class="lg:col-span-1">
  <div class="bg-base-100 rounded shadow h-full p-4">
    <h3 class="text-sm font-semibold mb-2 text-gray-600 dark:text-gray-300">
      Transaction History in 14 Days
    </h3>
    <TxsInBlocksChart />
      </div>
</div>


  </div>
</div>



<!-- Charts Section with top margin -->
    <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mt-6">
  <div class="bg-base-100 rounded p-4 shadow">
    <h3 class="font-semibold text-base mb-2">Daily User Transactions</h3>
    <v-chart class="h-72 w-full" :option="dailyUserTransactionsOption" autoresize />
  </div>

  <div class="bg-base-100 rounded p-4 shadow">
    <h3 class="font-semibold text-base mb-2">Network Fees (APT)</h3>
    <v-chart class="h-72 w-full" :option="networkFeesOption" autoresize />
  </div>
  </div>

  <div class="w-full">
  <div class="overflow-x-auto w-full mt-4 rounded bg-base-100 shadow">
    <div class="tabs tabs-boxed bg-transparent mb-4">
      <a
        class="tab text-gray-400 uppercase"
        :class="{ 'tab-active': tab === 'blocks' }"
        @click="tab = 'blocks'"
      >
        {{ $t('block') }}
      </a>
      <a
        class="tab text-gray-400 uppercase"
        :class="{ 'tab-active': tab === 'transactions' }"
        @click="tab = 'transactions'"
      >
        {{ $t('Transaction') }}
      </a>
    </div>

    <!-- BLOCKS -->
  <div v-show="tab === 'blocks'"  class="bg-base-100 rounded overflow-x-auto w-full">
  <table class="w-full table table-compact table-fixed">
    <thead>
    <tr class="bg-gray-50 dark:bg-[#1c2234] text-xs uppercase text-gray-500">
      <th class="p-3 w-[12%]">Block Height</th>
      <th class="p-3 w-[12%]">Age</th>
      <th class="p-3 w-[16%]">Generated By</th>
      <th class="p-3 w-[12%]">Transactions</th>
      <th class="p-3 w-[12%]">Round</th>
      <th class="p-3 w-[36%]">Hash</th>
    </tr>
      </thead>
    <tbody>
    <tr
      v-for="item in [...list].reverse().slice(0, 10)"
      :key="item.block.header.height"
      class="hover text-sm"
    >
      <td class="p-3">
        <RouterLink
          class="link link-primary no-underline"
          :to="`/${chain}/block/${item.block.header.height}`"
        >
          {{ item.block.header.height }}
        </RouterLink>
      </td>
      <td class="p-3">{{ format.toDay(item.block?.header?.time, 'from') }}</td>
      <td class="p-3">{{ format.validator(item.block?.header?.proposer_address) }}</td>
      <td class="p-3">{{ item.block?.data?.txs.length || 0 }}</td>
      <td class="p-3">{{ item.block?.last_commit?.round }}</td>
      <td class="p-3 text-xs text-gray-400 break-words whitespace-pre-wrap">
        {{ item.block_id.hash }}
      </td>
    </tr>
    </tbody>
  </table>


  <div class="flex justify-between items-center px-4 py-5 border-t mt-8">
    <div class="text-xs text-gray-400">
      {{ list.length.toLocaleString() }} Blocks
    </div>
    <RouterLink
      v-if="list.length > 10"
      to="/buycex/block"
      class="btn btn-sm bg-[#1a5fff] hover:bg-[#174ad1] text-white rounded px-4"
    >
      View All &nbsp; →
    </RouterLink>
  </div>
</div>


    <!-- TRANSACTIONS -->
    <div v-show="tab === 'transactions'" class="bg-base-100 rounded overflow-x-auto w-full">
      <table class="w-full table table-compact">
        <thead class="bg-gray-50 dark:bg-[#1c2234]">
          <tr>
            <th>{{ $t('account.height') }}</th>
            <th>{{ $t('account.hash') }}</th>
            <th>{{ $t('account.messages') }}</th>
            <th>{{ $t('block.fees') }}</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(item, index) in base.txsInRecents" :index="index" class="hover">
            <td class="text-sm text-primary">
              <RouterLink :to="`/${props.chain}/block/${item.height}`">{{ item.height }}</RouterLink>
            </td>
            <td class="truncate text-primary" width="50%">
              <RouterLink :to="`/${props.chain}/tx/${item.hash}`">{{ item.hash }}</RouterLink>
            </td>
            <td>{{ format.messages(item.tx.body.messages) }}</td>
            <td>{{ format.formatTokens(item.tx.authInfo.fee?.amount) }}</td>
          </tr>
        </tbody>
      </table>

      <div class="p-4">
        <div class="alert relative bg-transparent">
          <div class="alert absolute inset-x-0 inset-y-0 w-full h-full bg-info opacity-10"></div>
          <div class="text-info flex gap-2">
            <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24"
              class="stroke-current flex-shrink-0 w-6 h-6">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"></path>
            </svg>
            <span>{{ $t('block.only_tx') }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>


  

    <div class="bg-base-100 rounded mt-4 shadow">
      <div class="flex justify-between px-4 pt-4 pb-2 text-lg font-semibold text-main">
        <span class="truncate" >{{ walletStore.currentAddress || 'Not Connected' }}</span>
        <RouterLink v-if="walletStore.currentAddress"
          class="float-right text-sm cursor-pointert link link-primary no-underline font-medium"
          :to="`/${chain}/account/${walletStore.currentAddress}`">{{ $t('index.more') }}</RouterLink>
      </div>
      <div class="grid grid-cols-1 md:!grid-cols-4 auto-cols-auto gap-4 px-4 pb-6">
        <div class="bg-gray-100 dark:bg-[#373f59] rounded-sm px-4 py-3">
          <div class="text-sm mb-1">{{ $t('account.balance') }}</div>
          <div class="text-lg font-semibold text-main">
            {{ format.formatToken(walletStore.balanceOfStakingToken) }}
          </div>
          <div class="text-sm" :class="color">
            ${{ format.tokenValue(walletStore.balanceOfStakingToken) }}
          </div>
        </div>
        <div class="bg-gray-100 dark:bg-[#373f59] rounded-sm px-4 py-3">
          <div class="text-sm mb-1">{{ $t('module.staking') }}</div>
          <div class="text-lg font-semibold text-main">
            {{ format.formatToken(walletStore.stakingAmount) }}
          </div>
          <div class="text-sm" :class="color">
            ${{ format.tokenValue(walletStore.stakingAmount) }}
          </div>
        </div>
        <div class="bg-gray-100 dark:bg-[#373f59] rounded-sm px-4 py-3">
          <div class="text-sm mb-1">{{ $t('index.reward') }}</div>
          <div class="text-lg font-semibold text-main">
            {{ format.formatToken(walletStore.rewardAmount) }}
          </div>
          <div class="text-sm" :class="color">
            ${{ format.tokenValue(walletStore.rewardAmount) }}
          </div>
        </div>
        <div class="bg-gray-100 dark:bg-[#373f59] rounded-sm px-4 py-3">
          <div class="text-sm mb-1">{{ $t('index.unbonding') }}</div>
          <div class="text-lg font-semibold text-main">
            {{ format.formatToken(walletStore.unbondingAmount) }}
          </div>
          <div class="text-sm" :class="color">
            ${{ format.tokenValue(walletStore.unbondingAmount) }}
          </div>
        </div>
      </div>

      <div v-if="walletStore.delegations.length > 0" class="px-4 pb-4 overflow-auto">
        <table class="table table-compact w-full table-zebra">
          <thead>
            <tr>
              <th>{{ $t('account.validator') }}</th>
              <th>{{ $t('account.delegations') }}</th>
              <th>{{ $t('account.rewards') }}</th>
              <th>{{ $t('staking.actions') }}</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(item, index) in walletStore.delegations" :key="index">
              <td>
                <RouterLink class="link link-primary no-underline" :to="`/${chain}/staking/${item?.delegation?.validator_address}`">
                {{
                  format.validatorFromBech32(
                    item?.delegation?.validator_address
                  )
                }}
                </RouterLink>
              </td>
              <td>{{ format.formatToken(item?.balance) }}</td>
              <td>
                {{
                  format.formatTokens(
                    walletStore?.rewards?.rewards?.find(
                      (el) =>
                        el?.validator_address ===
                        item?.delegation?.validator_address
                    )?.reward)
                }}
              </td>
              <td>
                <div>
                  <label for="delegate" class="btn !btn-xs !btn-primary btn-ghost rounded-sm mr-2"
                    @click="dialog.open('delegate', { validator_address: item.delegation.validator_address }, updateState)">
                    {{ $t('account.btn_delegate') }}
                  </label>
                  <label for="withdraw" class="btn !btn-xs !btn-primary btn-ghost rounded-sm"
                    @click="dialog.open('withdraw', { validator_address: item.delegation.validator_address }, updateState)">
                    {{ $t('index.btn_withdraw_reward') }}
                  </label>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <div class="grid grid-cols-3 gap-4 px-4 pb-6 mt-4">
        <label for="PingTokenConvert" class="btn btn-primary text-white">{{ $t('index.btn_swap') }}</label>
        <label for="send" class="btn !bg-yes !border-yes text-white" @click="dialog.open('send', {}, updateState)">{{ $t('account.btn_send') }}</label>
        <label for="delegate" class="btn !bg-info !border-info text-white"
          @click="dialog.open('delegate', {}, updateState)">{{ $t('account.btn_delegate') }}</label>
        <RouterLink to="/wallet/receive" class="btn !bg-info !border-info text-white hidden">{{ $t('index.receive') }}</RouterLink>
      </div>
      <Teleport to="body">
        <ping-token-convert :chain-name="blockchain?.current?.prettyName" :endpoint="blockchain?.endpoint?.address"
          :hd-path="walletStore?.connectedWallet?.hdPath"></ping-token-convert>
      </Teleport>
    </div>

    <div class="bg-base-100 rounded mt-4">
    <div class="px-4 pt-4 pb-2 text-lg font-semibold text-main">
      {{ $t('index.app_versions') }}
    </div>
    <ArrayObjectElement :value="appVersionPatched" :thead="false" />
    <div class="h-4"></div>
  </div>

    <div v-if="!store.coingeckoId" class="bg-base-100 rounded mt-4">
      <div class="px-4 pt-4 pb-2 text-lg font-semibold text-main">
        {{ $t('index.node_info') }}
      </div>
      <ArrayObjectElement :value="paramStore.nodeVersion?.items" :thead="false" />      
      <div class="h-4"></div>
    </div>
  </div>
</template>

<route>
  {
    meta: {
      i18n: 'dashboard',
      order: 1,
    }
  }
</route>
