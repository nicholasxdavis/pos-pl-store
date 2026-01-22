<template>
  <div class="fill-height pa-8 d-flex flex-column align-center fade-in-up">
    <!-- Header -->
    <div class="w-100 max-width-600 mb-6 d-flex align-center">
        <v-btn icon="mdi-arrow-left" variant="text" class="mr-2" @click="$router.back()"></v-btn>
        <h2 class="text-h4 font-weight-light adaptive-text">Timer</h2>
    </div>

    <div class="max-width-600 w-100">
        <div class="glass-panel pa-6">
            <v-tabs v-model="tab" :color="themeColor" align-tabs="center" class="mb-8">
                <v-tab value="timer">Timer</v-tab>
                <v-tab value="stopwatch">Stopwatch</v-tab>
            </v-tabs>

            <v-window v-model="tab">
                <!-- TIMER TAB -->
                <v-window-item value="timer">
                    <div class="d-flex flex-column align-center">
                        
                        <!-- Circular Progress / Time Display -->
                        <div class="timer-circle mb-8 position-relative d-flex align-center justify-center">
                            <v-progress-circular
                                :model-value="timerProgress"
                                :color="timerColor"
                                :size="280"
                                :width="8"
                                class="timer-progress"
                            >
                                <div class="text-center">
                                    <div class="text-h2 font-weight-light adaptive-text font-monospace">{{ formatTime(timerRemaining) }}</div>
                                    <div class="text-caption opacity-60 mt-1">{{ timerState === 'running' ? 'Running' : (timerState === 'paused' ? 'Paused' : 'Set Time') }}</div>
                                </div>
                            </v-progress-circular>
                        </div>

                        <!-- Controls -->
                        <div class="d-flex align-center gap-4 mb-6">
                            <v-btn
                                v-if="timerState === 'idle'"
                                size="x-large"
                                icon="mdi-play"
                                :color="themeColor"
                                elevation="4"
                                @click="startTimer"
                            ></v-btn>
                             <v-btn
                                v-else-if="timerState === 'running'"
                                size="x-large"
                                icon="mdi-pause"
                                color="warning"
                                variant="tonal"
                                @click="pauseTimer"
                            ></v-btn>
                            <v-btn
                                v-else-if="timerState === 'paused'"
                                size="x-large"
                                icon="mdi-play"
                                color="success"
                                variant="tonal"
                                @click="resumeTimer"
                            ></v-btn>

                            <v-btn
                                v-if="timerState !== 'idle'"
                                size="large"
                                icon="mdi-refresh"
                                variant="text"
                                @click="resetTimer"
                            ></v-btn>
                        </div>

                        <!-- Time Input (Only when idle) -->
                        <div v-if="timerState === 'idle'" class="d-flex gap-2">
                             <v-chip-group v-model="selectedPreset" :color="themeColor">
                                <v-chip filter variant="outlined" :value="60" :color="themeColor">1m</v-chip>
                                <v-chip filter variant="outlined" :value="300" :color="themeColor">5m</v-chip>
                                <v-chip filter variant="outlined" :value="600" :color="themeColor">10m</v-chip>
                                <v-chip filter variant="outlined" :value="1500" :color="themeColor">25m</v-chip>
                            </v-chip-group>
                        </div>
                    </div>
                </v-window-item>

                <!-- STOPWATCH TAB -->
                <v-window-item value="stopwatch">
                     <div class="d-flex flex-column align-center">
                        <div class="text-h1 font-weight-light font-monospace mb-8 adaptive-text">
                            {{ formatStopwatch(stopwatchTime) }}
                        </div>

                         <div class="d-flex align-center gap-4 mb-8">
                            <v-btn
                                v-if="!stopwatchRunning"
                                size="x-large"
                                icon="mdi-play"
                                :color="themeColor"
                                @click="startStopwatch"
                            ></v-btn>
                             <v-btn
                                v-else
                                size="x-large"
                                icon="mdi-pause"
                                color="warning"
                                variant="tonal"
                                @click="stopStopwatch"
                            ></v-btn>

                             <v-btn
                                size="large"
                                icon="mdi-flag-outline"
                                variant="text"
                                :disabled="!stopwatchRunning"
                                @click="lapStopwatch"
                            ></v-btn>
                             <v-btn
                                size="large"
                                icon="mdi-refresh"
                                variant="text"
                                @click="resetStopwatch"
                            ></v-btn>
                        </div>
                        
                        <!-- Laps -->
                        <div class="w-100 overflow-y-auto custom-scrollbar" style="max-height: 200px">
                             <div 
                                v-for="(lap, index) in stopwatchLaps" 
                                :key="index"
                                class="d-flex justify-space-between py-2 px-4 border-bottom"
                            >
                                <span class="opacity-60">Lap {{ stopwatchLaps.length - index }}</span>
                                <span class="font-monospace">{{ formatStopwatch(lap) }}</span>
                            </div>
                        </div>

                    </div>
                </v-window-item>
            </v-window>
        </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, onUnmounted } from 'vue';

// Props from PluginView.vue
const props = defineProps({
    themeColor: { type: String, default: '#6366f1' },
    isDarkTheme: { type: Boolean, default: true }
});

// Use prop for theme color
const themeColor = props.themeColor;

const tab = ref('timer');

// --- TIMER LOGIC ---
const timerState = ref('idle');
const timerTotal = ref(300); // 5 mins default
const timerRemaining = ref(300);
const selectedPreset = ref(300);
let timerInterval = null;

const timerProgress = computed(() => {
    return (timerRemaining.value / timerTotal.value) * 100;
});

const timerColor = computed(() => {
    if (timerRemaining.value < 10) return 'error';
    if (timerRemaining.value < 60) return 'warning';
    return themeColor;
});

watch(selectedPreset, (val) => {
    if (val && timerState.value === 'idle') {
        timerTotal.value = val;
        timerRemaining.value = val;
    }
});

function startTimer() {
    timerState.value = 'running';
    timerInterval = setInterval(() => {
        if (timerRemaining.value > 0) {
            timerRemaining.value--;
        } else {
            timerFinished();
        }
    }, 1000);
}

function pauseTimer() {
    timerState.value = 'paused';
    if (timerInterval) clearInterval(timerInterval);
}

function resumeTimer() {
    timerState.value = 'running';
     timerInterval = setInterval(() => {
        if (timerRemaining.value > 0) {
            timerRemaining.value--;
        } else {
            timerFinished();
        }
    }, 1000);
}

function resetTimer() {
    pauseTimer();
    timerState.value = 'idle';
    timerRemaining.value = timerTotal.value;
}

function timerFinished() {
    pauseTimer();
    // Play sound (simple beep logic or audio file)
    playAlertSound();
    
    // Notification placeholder (plugin doesn't have access to app notification store)
    console.log('Timer finished!');
}

function formatTime(seconds) {
    const m = Math.floor(seconds / 60);
    const s = seconds % 60;
    return `${m.toString().padStart(2, '0')}:${s.toString().padStart(2, '0')}`;
}

// --- STOPWATCH LOGIC ---
const stopwatchRunning = ref(false);
const stopwatchTime = ref(0); // in ms
const stopwatchLaps = ref([]);
let stopwatchInterval = null;
let lastTime = 0;

function startStopwatch() {
    stopwatchRunning.value = true;
    lastTime = Date.now();
    
    stopwatchInterval = requestAnimationFrame(tickStopwatch);
}

function tickStopwatch() {
    if (!stopwatchRunning.value) return;
    
    const now = Date.now();
    const delta = now - lastTime;
    lastTime = now;
    stopwatchTime.value += delta;
    
    stopwatchInterval = requestAnimationFrame(tickStopwatch);
}

function stopStopwatch() {
    stopwatchRunning.value = false;
    if (stopwatchInterval) cancelAnimationFrame(stopwatchInterval);
}

function resetStopwatch() {
    stopStopwatch();
    stopwatchTime.value = 0;
    stopwatchLaps.value = [];
}

function lapStopwatch() {
    stopwatchLaps.value.unshift(stopwatchTime.value);
}

function formatStopwatch(ms) {
    const totalSeconds = Math.floor(ms / 1000);
    const m = Math.floor(totalSeconds / 60);
    const s = totalSeconds % 60;
    const cs = Math.floor((ms % 1000) / 10); // centiseconds
    return `${m.toString().padStart(2, '0')}:${s.toString().padStart(2, '0')}.${cs.toString().padStart(2, '0')}`;
}

function playAlertSound() {
    // Simple oscillator beep if simple
    // Or just rely on notification for now, or ensure we have an assets/sound beep
    const AudioContext = window.AudioContext || window.webkitAudioContext;
    if (AudioContext) {
        const ctx = new AudioContext();
        const osc = ctx.createOscillator();
        osc.connect(ctx.destination);
        osc.frequency.value = 880;
        osc.start();
        setTimeout(() => osc.stop(), 500);
    }
}

onUnmounted(() => {
    if (timerInterval) clearInterval(timerInterval);
    if (stopwatchInterval) cancelAnimationFrame(stopwatchInterval);
});

</script>

<style scoped>
.max-width-600 {
  max-width: 600px;
}

.font-monospace {
    font-family: 'Roboto Mono', monospace;
    font-variant-numeric: tabular-nums;
}

.gap-4 { gap: 16px; }
.gap-2 { gap: 8px; }

.timer-circle {
    filter: drop-shadow(0 0 15px rgba(var(--v-theme-primary), 0.2));
}

.border-bottom {
    border-bottom: 1px solid rgba(255, 255, 255, 0.05);
}

.custom-scrollbar::-webkit-scrollbar {
    width: 6px;
}
.custom-scrollbar::-webkit-scrollbar-thumb {
    background: rgba(255, 255, 255, 0.1);
    border-radius: 3px;
}
</style>
