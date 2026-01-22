<template>
    <div class="sports-widget d-flex flex-column h-100 w-100 text-left">
        <!-- Header -->
        <div class="d-flex align-center justify-space-between mb-2">
            <div class="align-self-start">
                <div 
                    class="text-overline adaptive-text opacity-80 mb-1"
                    :style="{ color: isDarkTheme ? 'inherit' : '#333333' }"
                >SPORTS</div>
            </div>
        </div>

        <!-- Content -->
        <div 
            class="d-flex flex-grow-1" 
            :class="size === 'square' ? 'flex-column align-center justify-center text-center' : 'align-center overflow-x-auto hide-scrollbar'"
        >
            <!-- Wide Mode: 6 Games -->
            <div v-if="size === 'wide'" class="d-flex align-center gap-4 px-2 w-100 justify-space-around">
            <div v-for="(game, i) in sportsData.slice(0, 6)" :key="i" class="game-card d-flex flex-column align-center">
                <div class="d-flex align-center justify-center gap-2">
                    <!-- Team 1 -->
                    <div class="team-block">
                        <img :src="game.logo1" class="team-logo" :alt="game.team1" />
                        <div class="score">{{ game.score1 }}</div>
                    </div>
                    
                    <!-- VS -->
                    <div class="vs-text">vs</div>
                    
                    <!-- Team 2 -->
                    <div class="team-block">
                        <img :src="game.logo2" class="team-logo" :alt="game.team2" />
                        <div class="score">{{ game.score2 }}</div>
                    </div>
                </div>
                <div class="league-badge">{{ game.league }}</div>
            </div>
        </div>

        <!-- Square Mode: 1 Game -->
        <div v-else class="d-flex flex-column align-center justify-center text-center w-100 pa-2">
            <div class="d-flex align-center justify-center gap-4 mb-2">
                <!-- Team 1 -->
                <div class="team-block-large">
                    <img :src="currentGame.logo1" class="team-logo-large" :alt="currentGame.team1" />
                    <div class="score-large">{{ currentGame.score1 }}</div>
                </div>
                
                <!-- VS -->
                <div class="vs-text-large">vs</div>
                
                <!-- Team 2 -->
                <div class="team-block-large">
                    <img :src="currentGame.logo2" class="team-logo-large" :alt="currentGame.team2" />
                    <div class="score-large">{{ currentGame.score2 }}</div>
                </div>
            </div>
            <div class="text-caption opacity-60">{{ currentGame.league }}</div>
        </div>
        </div>
    </div>
</template>

<script setup>
import { computed, ref, onMounted, onUnmounted } from 'vue';

const sportsData = [
    { league: 'NBA', team1: 'LAL', team2: 'GSW', score1: '102', score2: '98', logo1: 'https://a.espncdn.com/i/teamlogos/nba/500/lal.png', logo2: 'https://a.espncdn.com/i/teamlogos/nba/500/gsw.png' },
    { league: 'NFL', team1: 'KC', team2: 'BUF', score1: '24', score2: '21', logo1: 'https://a.espncdn.com/i/teamlogos/nfl/500/kc.png', logo2: 'https://a.espncdn.com/i/teamlogos/nfl/500/buf.png' },
    { league: 'MLB', team1: 'NYY', team2: 'BOS', score1: '5', score2: '3', logo1: 'https://a.espncdn.com/i/teamlogos/mlb/500/nyy.png', logo2: 'https://a.espncdn.com/i/teamlogos/mlb/500/bos.png' },
    { league: 'NHL', team1: 'BOS', team2: 'TOR', score1: '4', score2: '2', logo1: 'https://a.espncdn.com/i/teamlogos/nhl/500/bos.png', logo2: 'https://a.espncdn.com/i/teamlogos/nhl/500/tor.png' },
    { league: 'NBA', team1: 'PHX', team2: 'DAL', score1: '115', score2: '110', logo1: 'https://a.espncdn.com/i/teamlogos/nba/500/phx.png', logo2: 'https://a.espncdn.com/i/teamlogos/nba/500/dal.png' },
    { league: 'NFL', team1: 'SF', team2: 'DET', score1: '34', score2: '31', logo1: 'https://a.espncdn.com/i/teamlogos/nfl/500/sf.png', logo2: 'https://a.espncdn.com/i/teamlogos/nfl/500/det.png' }
];

const props = defineProps({
    size: { type: String, default: 'square' },
    themeColor: String,
    isDarkTheme: Boolean
});

const currentIndex = ref(0);
const currentGame = computed(() => sportsData[currentIndex.value]);
let timer = null;

onMounted(() => {
    timer = setInterval(() => {
        currentIndex.value = (currentIndex.value + 1) % sportsData.length;
    }, 4000);
});

onUnmounted(() => {
    if (timer) clearInterval(timer);
});
</script>

<style scoped>
.gap-4 { gap: 16px; }
.gap-2 { gap: 8px; }

.hide-scrollbar {
    -ms-overflow-style: none;
    scrollbar-width: none;
}
.hide-scrollbar::-webkit-scrollbar {
    display: none;
}

.game-card {
    min-width: 90px;
}

.team-block {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 2px;
}

.team-block-large {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 4px;
}

.team-logo {
    width: 24px;
    height: 24px;
    object-fit: contain;
}

.team-logo-large {
    width: 40px;
    height: 40px;
    object-fit: contain;
}

.score {
    font-size: 0.75rem;
    font-weight: 700;
}

.score-large {
    font-size: 1.25rem;
    font-weight: 700;
}

.vs-text {
    font-size: 0.6rem;
    opacity: 0.4;
    font-weight: 300;
}

.vs-text-large {
    font-size: 0.875rem;
    opacity: 0.4;
    font-weight: 300;
}

.league-badge {
    font-size: 0.6rem;
    opacity: 0.5;
    margin-top: 4px;
    text-transform: uppercase;
    letter-spacing: 0.5px;
}
</style>
