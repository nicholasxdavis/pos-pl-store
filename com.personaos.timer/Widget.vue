<template>
    <div class="timer-widget d-flex flex-column h-100 w-100 text-left">
        <!-- Header -->
        <div class="d-flex align-center justify-space-between mb-2">
            <div class="align-self-start">
                <div 
                    class="text-overline adaptive-text opacity-80 mb-1"
                    :style="{ color: isDarkTheme ? 'inherit' : '#333333' }"
                >TIMER</div>
            </div>
        </div>

        <!-- Time Display -->
        <div class="d-flex align-center justify-center flex-grow-1">
            <div 
                class="time-display"
                :class="{ 
                    'text-h3': size === 'wide', 
                    'text-h4': size === 'square',
                    'adaptive-text': isDarkTheme
                }"
                :style="{ color: isDarkTheme ? 'inherit' : '#333333' }"
            >
                {{ formattedTime }}
            </div>
        </div>

        <!-- Controls -->
        <div class="d-flex align-center justify-center gap-2" :class="{ 'mt-2': size === 'square' }">
            <v-btn 
                v-if="!isRunning"
                icon="mdi-play" 
                variant="text" 
                :color="themeColor" 
                :size="size === 'wide' ? 'default' : 'small'"
                @click="startTimer"
            ></v-btn>
            <v-btn 
                v-else
                icon="mdi-pause" 
                variant="text" 
                :color="themeColor" 
                :size="size === 'wide' ? 'default' : 'small'"
                @click="pauseTimer"
            ></v-btn>
            <v-btn 
                icon="mdi-stop" 
                variant="text" 
                :color="themeColor" 
                :size="size === 'wide' ? 'default' : 'small'"
                :disabled="!isRunning && elapsedTime === 0"
                @click="resetTimer"
            ></v-btn>
        </div>
    </div>
</template>

<script setup>
import { ref, computed, onUnmounted } from 'vue';

const props = defineProps({
    size: { type: String, default: 'square' },
    themeColor: String,
    isDarkTheme: Boolean
});

const isRunning = ref(false);
const elapsedTime = ref(0); // in seconds
let intervalId = null;

const formattedTime = computed(() => {
    const hours = Math.floor(elapsedTime.value / 3600);
    const minutes = Math.floor((elapsedTime.value % 3600) / 60);
    const seconds = elapsedTime.value % 60;
    
    if (hours > 0) {
        return `${String(hours).padStart(2, '0')}:${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`;
    }
    return `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`;
});

const startTimer = () => {
    isRunning.value = true;
    intervalId = setInterval(() => {
        elapsedTime.value++;
    }, 1000);
};

const pauseTimer = () => {
    isRunning.value = false;
    if (intervalId) {
        clearInterval(intervalId);
        intervalId = null;
    }
};

const resetTimer = () => {
    pauseTimer();
    elapsedTime.value = 0;
};

onUnmounted(() => {
    if (intervalId) {
        clearInterval(intervalId);
    }
});
</script>

<style scoped>
.timer-widget {
    display: flex;
    flex-direction: column;
    height: 100%;
}

.time-display {
    font-family: 'Geist Mono', monospace;
    letter-spacing: 2px;
    line-height: 1;
}

.gap-2 {
    gap: 8px;
}
</style>
