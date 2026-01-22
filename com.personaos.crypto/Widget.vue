<template>
    <div 
        class="d-flex h-100 w-100" 
        :class="size === 'square' ? 'flex-column align-center justify-center text-center' : 'align-center overflow-x-auto hide-scrollbar'"
    >
        <!-- Wide Mode: 6 Coins -->
        <div v-if="size === 'wide'" class="d-flex align-center gap-4 px-2 w-100 justify-space-around">
            <div v-for="(coin, i) in cryptoData.slice(0, 6)" :key="i" class="coin-card d-flex flex-column align-center">
                <img :src="coin.icon" class="coin-logo" :alt="coin.symbol" />
                <div class="coin-symbol">{{ coin.symbol }}</div>
                <div class="coin-price">${{ coin.price }}</div>
                <div class="coin-change" :class="coin.change >= 0 ? 'positive' : 'negative'">
                    {{ coin.change >= 0 ? '▲' : '▼' }} {{ Math.abs(coin.change) }}%
                </div>
            </div>
        </div>

        <!-- Square Mode: 1 Coin -->
        <div v-else class="d-flex flex-column align-center justify-center text-center w-100 pa-2">
            <img :src="currentCrypto.icon" class="coin-logo-large" :alt="currentCrypto.symbol" />
            <div class="coin-symbol-large">{{ currentCrypto.symbol }}</div>
            <div class="coin-name">{{ currentCrypto.name }}</div>
            <div class="coin-price-large">${{ currentCrypto.price }}</div>
            <div class="coin-change-large" :class="currentCrypto.change >= 0 ? 'positive' : 'negative'">
                {{ currentCrypto.change >= 0 ? '▲' : '▼' }} {{ Math.abs(currentCrypto.change) }}%
            </div>
        </div>
    </div>
</template>

<script setup>
import { computed, ref, onMounted, onUnmounted } from 'vue';

// Mix of top gainers and losers with real icons
const cryptoData = [
    { symbol: 'BTC', name: 'Bitcoin', price: '43,250', change: 2.4, icon: 'https://assets.coincap.io/assets/icons/btc@2x.png' },
    { symbol: 'ETH', name: 'Ethereum', price: '2,280', change: -1.2, icon: 'https://assets.coincap.io/assets/icons/eth@2x.png' },
    { symbol: 'SOL', name: 'Solana', price: '98.50', change: 5.7, icon: 'https://assets.coincap.io/assets/icons/sol@2x.png' },
    { symbol: 'XRP', name: 'Ripple', price: '0.62', change: 3.2, icon: 'https://assets.coincap.io/assets/icons/xrp@2x.png' },
    { symbol: 'DOGE', name: 'Dogecoin', price: '0.08', change: -3.1, icon: 'https://assets.coincap.io/assets/icons/doge@2x.png' },
    { symbol: 'ADA', name: 'Cardano', price: '0.52', change: 1.5, icon: 'https://assets.coincap.io/assets/icons/ada@2x.png' }
];

const props = defineProps({
    size: { type: String, default: 'square' },
    themeColor: String,
    isDarkTheme: Boolean
});

const currentIndex = ref(0);
const currentCrypto = computed(() => cryptoData[currentIndex.value]);
let timer = null;

onMounted(() => {
    timer = setInterval(() => {
        currentIndex.value = (currentIndex.value + 1) % cryptoData.length;
    }, 4000);
});

onUnmounted(() => {
    if (timer) clearInterval(timer);
});
</script>

<style scoped>
.gap-4 { gap: 16px; }

.hide-scrollbar {
    -ms-overflow-style: none;
    scrollbar-width: none;
}
.hide-scrollbar::-webkit-scrollbar {
    display: none;
}

.coin-card {
    min-width: 70px;
}

.coin-logo {
    width: 28px;
    height: 28px;
    object-fit: contain;
    border-radius: 50%;
    margin-bottom: 4px;
}

.coin-logo-large {
    width: 48px;
    height: 48px;
    object-fit: contain;
    border-radius: 50%;
    margin-bottom: 6px;
}

.coin-symbol {
    font-size: 0.7rem;
    font-weight: 700;
    letter-spacing: 0.5px;
}

.coin-symbol-large {
    font-size: 1rem;
    font-weight: 700;
    letter-spacing: 0.5px;
}

.coin-name {
    font-size: 0.7rem;
    opacity: 0.5;
    margin-bottom: 4px;
}

.coin-price {
    font-size: 0.65rem;
    opacity: 0.8;
}

.coin-price-large {
    font-size: 1.25rem;
    font-weight: 600;
    margin-bottom: 2px;
}

.coin-change {
    font-size: 0.6rem;
    font-weight: 600;
}

.coin-change-large {
    font-size: 0.875rem;
    font-weight: 600;
}

.positive {
    color: #4CAF50;
}

.negative {
    color: #FF5252;
}
</style>
