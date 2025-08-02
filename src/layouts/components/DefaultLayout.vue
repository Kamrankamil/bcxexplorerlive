<script lang="ts" setup>
import { Icon } from '@iconify/vue';
import { computed, ref } from 'vue';

// Components
import newFooter from '@/layouts/components/NavFooter.vue';
import NavbarThemeSwitcher from '@/layouts/components/NavbarThemeSwitcher.vue';
import NavbarSearch from '@/layouts/components/NavbarSearch.vue';
import ChainProfile from '@/layouts/components/ChainProfile.vue';

import { NetworkType, useDashboard } from '@/stores/useDashboard';
import { useBaseStore, useBlockchain } from '@/stores';

import NavBarI18n from './NavBarI18n.vue';
import NavBarWallet from './NavBarWallet.vue';
import type { NavGroup, NavLink, NavSectionTitle, VerticalNavItems } from '../types';
import dayjs from 'dayjs';
import AdBanner from '@/components/ad/AdBanner.vue';

const dashboard = useDashboard();
dashboard.initial();
const blockchain = useBlockchain();
blockchain.randomSetupEndpoint();
const baseStore = useBaseStore();

const current = ref('');
const temp = ref('')
blockchain.$subscribe((m, s) => {
  if (current.value === s.chainName && temp.value != s.endpoint.address) {
    temp.value = s.endpoint.address
    blockchain.initial();
  }
  if (current.value != s.chainName) {
    current.value = s.chainName;
    blockchain.randomSetupEndpoint();
  }
});

const sidebarShow = ref(false);
const sidebarOpen = ref(true);

const changeOpen = (index: Number) => {
  if (index === 0) {
    sidebarOpen.value = !sidebarOpen.value;
  }
};
const showDiscord = window.location.host.search('buycex') > -1;

function isNavGroup(nav: VerticalNavItems | any): nav is NavGroup {
  return (<NavGroup>nav).children !== undefined;
}
function isNavLink(nav: VerticalNavItems | any): nav is NavLink {
  return (<NavLink>nav).to !== undefined;
}
function isNavTitle(nav: VerticalNavItems | any): nav is NavSectionTitle {
  return (<NavSectionTitle>nav).heading !== undefined;
}
function selected(route: any, nav: NavLink) {
  const b = route.path === nav.to?.path || route.path.startsWith(nav.to?.path) && nav.title.indexOf('dashboard') === -1
  return b
}
const blocktime = computed(() => {
  return dayjs(baseStore.latest?.block?.header?.time)
});

const behind = computed(() => {
  const current = dayjs().subtract(10, 'minute')
  return blocktime.value.isBefore(current)
});

dayjs()

const show_ad = computed(() => {
  return location.hostname.indexOf('buycex') > -1
})

</script>

<template>
  <div class="bg-gray-100 dark:bg-[#171d30]">
    <!-- Horizontal Navbar -->
    <div class="bg-base-100 border-b border-gray-100 dark:border-gray-700 px-6 py-3 flex items-center justify-between">
      <!-- Logo -->
      <RouterLink to="/" class="flex items-center gap-3">
      <ChainProfile class="mb-4" />
        <h1 class="text-2xl font-semibold dark:text-white">Buycex</h1>
      </RouterLink>

      <!-- Flat Menu -->
      <div class="flex gap-6 items-center overflow-x-auto">
        <template
          v-for="(item, index) in blockchain.computedChainMenu.filter(i =>
            !['ecosystem', 'favorite', 'allblockchains', 'tools', 'wallethelper', 'sponsors', 'cosmoshub', 'osmosis', 'celestia']
              .includes(i.title?.toLowerCase()?.replace(/\s+/g, ''))
          )"
          :key="index"
        >
          <template v-if="isNavGroup(item)">
            <RouterLink
              v-for="(el, key) in item.children"
              :key="key"
              :to="el.to"
              class="text-gray-700 dark:text-gray-200 hover:text-primary text-sm capitalize"
              :class="{ 'font-semibold text-primary': selected($route, el) }"
            >
              {{ $t(el.title) }}
            </RouterLink>
          </template>
          <RouterLink
            v-if="isNavLink(item)"
            :to="item.to"
            class="text-gray-700 dark:text-gray-200 hover:text-primary text-sm capitalize"
            :class="{ 'font-semibold text-primary': selected($route, item) }"
          >
            {{ item.title }}
          </RouterLink>
        </template>
      </div>

      <!-- Right: Search, Theme, Language, Wallet -->
      <div class="flex items-center gap-4">
        <NavbarSearch />
        <NavbarThemeSwitcher />
        <NavBarI18n />
        <NavBarWallet />
      </div>
    </div>

  

    <div class="px-6 pt-6">
     

      <div v-if="behind" class="alert alert-error mb-4">
        <div class="flex gap-2">
          <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24"
            class="stroke-current flex-shrink-0 w-6 h-6">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
              d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"></path>
          </svg>
          <span>{{ $t('pages.out_of_sync') }} {{ blocktime.format() }} ({{ blocktime.fromNow() }})</span>
        </div>
      </div>

      <RouterView v-slot="{ Component }">
        <Transition mode="out-in">
          <div>
            <AdBanner v-if="show_ad" />
            <Component :is="Component" />
          </div>
        </Transition>
      </RouterView>

      <newFooter />
    </div>
  </div>
</template>

