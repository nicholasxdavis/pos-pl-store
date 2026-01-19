<template>
    <div 
        class="d-flex h-100 w-100" 
        :class="tall ? 'flex-column align-center justify-center text-center' : 'align-center overflow-x-auto hide-scrollbar'"
    >
        <!-- Wide Mode: Multiple Cryptos -->
        <div v-if="!tall" class="d-flex align-center gap-6 px-2 w-100 justify-space-between">
            <div v-for="(coin, i) in cryptoData" :key="i" class="d-flex flex-column align-center" style="min-width: 80px;">
                <img :src="coin.icon" height="28" width="28" class="crypto-logo mb-1" :alt="coin.symbol" />
                <div class="text-subtitle-2 font-weight-bold adaptive-text">{{ coin.symbol }}</div>
                <div class="text-caption font-weight-medium">${{ coin.price }}</div>
                <div class="text-caption" :class="coin.change >= 0 ? 'text-green' : 'text-red'" style="font-size: 0.7rem;">
                    {{ coin.change >= 0 ? '+' : '' }}{{ coin.change }}%
                </div>
            </div>
        </div>

        <!-- Tall Mode: Single Crypto -->
        <div v-else class="d-flex flex-column align-center justify-center text-center w-100">
            <div class="icon-container mb-3" :class="{ 'dark-theme': isDarkTheme }">
                <img :src="currentCrypto.icon" height="36" width="36" class="crypto-logo" :alt="currentCrypto.symbol" />
            </div>
            <div class="text-center">
                <div class="text-subtitle-1 font-weight-medium adaptive-text">{{ currentCrypto.symbol }}</div>
                <div class="text-caption adaptive-text opacity-60">{{ currentCrypto.name }}</div>
            </div>
            <div class="mt-1">
                <div class="text-h6 adaptive-text font-weight-bold">${{ currentCrypto.price }}</div>
                <div class="text-caption" :class="currentCrypto.change >= 0 ? 'text-success' : 'text-error'">
                    {{ currentCrypto.change >= 0 ? '+' : '' }}{{ currentCrypto.change }}%
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { computed, ref, onMounted, onUnmounted } from 'vue';

const cryptoData = [
    { symbol: 'BTC', name: 'Bitcoin', price: '43,250', change: 2.4, icon: 'https://assets.coincap.io/assets/icons/btc@2x.png' },
    { symbol: 'ETH', name: 'Ethereum', price: '2,280', change: -1.2, icon: 'https://assets.coincap.io/assets/icons/eth@2x.png' },
    { symbol: 'SOL', name: 'Solana', price: '98.50', change: 5.7, icon: 'https://assets.coincap.io/assets/icons/sol@2x.png' },
    { symbol: 'AAPL', name: 'Apple Inc.', price: '185.20', change: 0.8, icon: 'https://logo.clearbit.com/apple.com' },
    { symbol: 'DOGE', name: 'Dogecoin', price: '0.08', change: -3.1, icon: 'https://assets.coincap.io/assets/icons/doge@2x.png' },
    { symbol: 'ADA', name: 'Cardano', price: '0.52', change: 1.5, icon: 'https://assets.coincap.io/assets/icons/ada@2x.png' }
];

const props = defineProps({
    tall: Boolean,
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
.gap-6 { gap: 24px; }
.hide-scrollbar {
    -ms-overflow-style: none;
    scrollbar-width: none;
}
.hide-scrollbar::-webkit-scrollbar {
    display: none;
}
.icon-container {
    width: 56px;
    height: 56px;
    border-radius: 12px;
    display: flex;
    align-items: center;
    justify-content: center;
    background: rgba(200, 200, 200, 0.15);
}
.icon-container.dark-theme {
    background: color-mix(in srgb, var(--theme-color), transparent 70%);
}
.text-success { color: #4CAF50 !important; }
.text-error { color: #FF5252 !important; }
.crypto-logo {
    object-fit: contain;
    border-radius: 50%;
}
</style>
