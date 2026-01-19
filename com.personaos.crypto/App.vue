<template>
  <div class="fill-height pa-8 d-flex flex-column fade-in-up">
    <!-- Header -->
    <div class="d-flex align-center justify-space-between mb-6">
      <div class="d-flex align-center">
        <v-btn icon="mdi-arrow-left" variant="text" class="mr-2" @click="$router.back()"></v-btn>
        <h2 class="text-h4 font-weight-light adaptive-text">Crypto Plugin</h2>
      </div>
      <div class="d-flex gap-2">
        <v-btn icon="mdi-refresh" variant="text" @click="refreshData"></v-btn>
      </div>
    </div>

    <!-- Category Pills -->
    <div class="mb-6 d-flex overflow-x-auto custom-scrollbar gap-2" :class="{'opacity-50': loading}">
      <v-chip
        v-for="cat in categories"
        :key="cat.id"
        :color="currentCategory === cat.id ? uiStore.themeColor : 'default'"
        :variant="currentCategory === cat.id ? 'flat' : 'outlined'"
        class="text-uppercase"
        style="min-width: 80px; justify-content: center;"
        @click="currentCategory = cat.id"
      >
        {{ cat.label }}
      </v-chip>
    </div>

    <!-- Loading State -->
    <div v-if="loading" class="d-flex justify-center align-center flex-grow-1">
      <v-progress-circular indeterminate :color="uiStore.themeColor" size="64"></v-progress-circular>
    </div>

    <!-- List View -->
    <div v-else class="overflow-y-auto custom-scrollbar flex-grow-1">
      <div class="glass-panel pa-0 overflow-hidden">
        <!-- List Header -->
         <div class="d-flex align-center px-4 py-3 bg-white-5 text-caption opacity-60">
             <div style="width: 40px"></div> <!-- Icon space -->
             <div class="flex-grow-1">Name</div>
             <div style="width: 80px" class="text-right">Price</div>
             <div style="width: 80px" class="text-right">24h %</div>
             <div style="width: 20px"></div> <!-- Chevron space -->
         </div>
         <v-divider class="opacity-10"></v-divider>

         <!-- Items -->
         <div 
            v-for="(item, index) in filteredItems" 
            :key="item.id" 
            class="d-flex align-center px-4 py-3 cursor-pointer hover-bg"
            :class="{ 'border-bottom': index < filteredItems.length - 1 }"
            @click="showDetail(item)"
         >
             <!-- Icon -->
             <v-avatar size="32" class="mr-3 bg-white-5">
                 <v-img v-if="item.image" :src="item.image"></v-img>
                 <span v-else class="text-caption font-weight-bold">{{ item.symbol.slice(0, 2) }}</span>
             </v-avatar>

             <!-- Name & Symbol -->
             <div class="flex-grow-1 d-flex flex-column" style="min-width: 0">
                 <span class="font-weight-medium text-truncate">{{ item.name }}</span>
                 <span class="text-caption opacity-50">{{ item.symbol }}</span>
             </div>

             <!-- Price -->
             <div class="text-right font-monospace font-weight-medium" style="width: 80px">
                 {{ formatCurrency(item.current_price) }}
             </div>

             <!-- Change -->
             <div class="text-right font-monospace font-weight-bold" style="width: 80px" :class="getChangeColor(item.price_change_percentage_24h)">
                 {{ item.price_change_percentage_24h > 0 ? '+' : '' }}{{ item.price_change_percentage_24h?.toFixed(2) }}%
             </div>

             <!-- Chevron -->
             <v-icon class="ml-4 opacity-30" size="small">mdi-chevron-right</v-icon>
         </div>
      </div>
    </div>

    <!-- Detail Dialog -->
    <v-dialog v-model="dialog" max-width="600">
      <v-card v-if="selectedItem" class="pa-2">
         <v-card-title class="d-flex justify-space-between align-center">
             <div class="d-flex align-center">
                 <v-avatar size="32" class="mr-3 bg-white-5">
                     <v-img v-if="selectedItem.image" :src="selectedItem.image"></v-img>
                     <span v-else class="text-caption">{{ selectedItem.symbol }}</span>
                 </v-avatar>
                 <span>{{ selectedItem.name }} ({{ selectedItem.symbol }})</span>
             </div>
             <v-btn icon="mdi-close" variant="text" size="small" @click="dialog = false"></v-btn>
         </v-card-title>

         <v-card-text class="pt-4">
             <!-- Big Price -->
             <div class="text-center mb-6">
                 <div class="text-h3 font-weight-bold mb-1">{{ formatCurrency(selectedItem.current_price) }}</div>
                 <div class="text-h6" :class="getChangeColor(selectedItem.price_change_percentage_24h)">
                     {{ selectedItem.price_change_percentage_24h > 0 ? '+' : '' }}{{ selectedItem.price_change_percentage_24h?.toFixed(2) }}%
                 </div>
             </div>

             <!-- Chart Placeholder -->
             <div class="glass-panel pa-4 mb-4" style="height: 200px; position: relative;">
                <Line
                    v-if="chartData"
                    :data="chartData"
                    :options="chartOptions"
                />
                <div v-else class="d-flex justify-center align-center fill-height opacity-50">
                    <v-progress-circular indeterminate size="32" v-if="loadingChart"></v-progress-circular>
                    <span v-else>No chart data available</span>
                </div>
             </div>

             <!-- Stats Grid -->
             <v-row>
                 <v-col cols="6">
                     <div class="text-caption opacity-60">Market Cap</div>
                     <div class="font-weight-medium">{{ formatCurrency(selectedItem.market_cap, true) }}</div>
                 </v-col>
                 <v-col cols="6">
                     <div class="text-caption opacity-60">Volume (24h)</div>
                     <div class="font-weight-medium">{{ formatCurrency(selectedItem.total_volume, true) }}</div>
                 </v-col>
                 <v-col cols="6">
                     <div class="text-caption opacity-60">High (24h)</div>
                     <div class="font-weight-medium">{{ formatCurrency(selectedItem.high_24h) }}</div>
                 </v-col>
                 <v-col cols="6">
                     <div class="text-caption opacity-60">Low (24h)</div>
                     <div class="font-weight-medium">{{ formatCurrency(selectedItem.low_24h) }}</div>
                 </v-col>
             </v-row>
         </v-card-text>
      </v-card>
    </v-dialog>

  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue';
import { useUIStore } from '@/stores/ui';
import { API_KEYS } from '@/config/keys';
import {
  Chart as ChartJS,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Filler,
} from 'chart.js';
import { Line } from 'vue-chartjs';

ChartJS.register(
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Filler
);

const uiStore = useUIStore();

const categories = [
    { id: 'crypto', label: 'Crypto' },
    { id: 'stocks', label: 'Stocks' }
];

const currentCategory = ref('crypto');
const loading = ref(false);
const cryptoData = ref([]);
const stockData = ref([]); // Mocked or limited for now
const dialog = ref(false);
const selectedItem = ref(null);
const chartData = ref(null);
const loadingChart = ref(false);

const filteredItems = computed(() => {
    return currentCategory.value === 'crypto' ? cryptoData.value : stockData.value;
});

// CoinGecko for Crypto
async function fetchCrypto() {
    try {
        const res = await fetch('https://api.coingecko.com/api/v3/coins/markets?vs_currency=usd&order=market_cap_desc&per_page=50&page=1&sparkline=false');
        if (!res.ok) throw new Error('Rate Limit');
        const data = await res.json();
        return data.map(c => ({
            id: c.id,
            name: c.name,
            symbol: c.symbol.toUpperCase(),
            image: c.image,
            current_price: c.current_price,
            price_change_percentage_24h: c.price_change_percentage_24h,
            market_cap: c.market_cap,
            total_volume: c.total_volume,
            high_24h: c.high_24h,
            low_24h: c.low_24h,
            type: 'crypto'
        }));
    } catch (e) {
        console.warn('Crypto fetch failed', e);
        return [];
    }
}

// Finnhub for Stocks (Limited)
// Since Finnhub is per-symbol, we pick a few popular ones
const POPULAR_STOCKS = ['AAPL', 'MSFT', 'GOOGL', 'AMZN', 'TSLA', 'NVDA', 'META'];
async function fetchStocks() {
    // Note: Free tier Finnhub has limits, so we cache or fetch sequentially carefully
    // For demo/stability, we might fetch just quotes
    const promises = POPULAR_STOCKS.map(async (sym) => {
        try {
            const res = await fetch(`https://finnhub.io/api/v1/quote?symbol=${sym}&token=${API_KEYS.FINNHUB_KEY}`);
            const data = await res.json();
            return {
                id: sym,
                name: sym, // Finnhub quote doesn't give name, usually need profile. keeping simple.
                symbol: sym,
                image: `https://logo.clearbit.com/${sym.toLowerCase()}.com`, // clearbit fallback for logos
                current_price: data.c,
                price_change_percentage_24h: data.dp,
                high_24h: data.h,
                low_24h: data.l,
                type: 'stock'
            };
        } catch { return null; }
    });
    const results = await Promise.all(promises);
    return results.filter(r => r !== null);
}

async function refreshData() {
    loading.value = true;
    const [cData, sData] = await Promise.all([fetchCrypto(), fetchStocks()]);
    cryptoData.value = cData;
    stockData.value = sData;
    loading.value = false;
}

// Chart Logic
async function loadChart(item) {
    loadingChart.value = true;
    chartData.value = null;
    
    try {
        let prices = [];
        let labels = [];

        if (item.type === 'crypto') {
            const res = await fetch(`https://api.coingecko.com/api/v3/coins/${item.id}/market_chart?vs_currency=usd&days=7`);
            const data = await res.json();
            prices = data.prices.map(p => p[1]);
            labels = data.prices.map(p => new Date(p[0]).toLocaleDateString());
        } else {
            // Mock stock history or use candles endpoint if available in key plan
            // Finnhub candles: /stock/candle?symbol=AAPL&resolution=D&from=...&to=...
            // Fallback mock for demo
             prices = Array(7).fill(0).map((_, i) => item.current_price * (1 + (Math.random() * 0.1 - 0.05)));
             labels = ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun'];
        }

        const color = uiStore.themeColor || '#ffffff';
        
        chartData.value = {
            labels: labels,
            datasets: [{
                label: 'Price (7d)',
                backgroundColor: color + '20', // transparent
                borderColor: color,
                data: prices,
                fill: true,
                tension: 0.4,
                pointRadius: 0
            }]
        };

    } catch (e) {
        console.error('Chart load error', e);
    }
    loadingChart.value = false;
}

const chartOptions = computed(() => ({
    responsive: true,
    maintainAspectRatio: false,
    plugins: {
        legend: { display: false },
        tooltip: {
             mode: 'index',
             intersect: false,
        }
    },
    scales: {
        x: { display: false },
        y: { display: false }
    }
}));

function showDetail(item) {
    selectedItem.value = item;
    dialog.value = true;
    loadChart(item);
}

function formatCurrency(val, compact = false) {
    if (val === undefined || val === null) return '-';
    return new Intl.NumberFormat('en-US', {
        style: 'currency',
        currency: 'USD',
        notation: compact ? 'compact' : 'standard'
    }).format(val);
}

function getChangeColor(val) {
    if (!val) return '';
    return val > 0 ? 'text-success' : val < 0 ? 'text-error' : '';
}

onMounted(() => {
    refreshData();
});
</script>

<style scoped>
.gap-2 { gap: 8px; }
.hover-bg { transition: background-color 0.2s ease; }
.hover-bg:hover { background: rgba(255, 255, 255, 0.05); }
.border-bottom { border-bottom: 1px solid rgba(255, 255, 255, 0.1); }
.bg-white-5 { background: rgba(255,255,255,0.05); }
.font-monospace { font-family: 'Geist Mono', monospace; }
</style>
