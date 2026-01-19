<template>
    <div 
        class="d-flex h-100 w-100" 
        :class="tall ? 'flex-column align-center justify-center text-center' : 'align-center overflow-x-auto hide-scrollbar'"
    >
        <!-- Wide Mode: Multiple Games -->
        <div v-if="!tall" class="d-flex align-center gap-6 px-2 w-100 justify-space-between">
            <div v-for="(game, i) in sportsData" :key="i" class="d-flex flex-column align-center" style="min-width: 100px;">
                <div class="d-flex align-center justify-center mb-1 w-100">
                    <div class="d-flex flex-column align-center mx-2">
                        <img :src="game.logo1" height="24" width="24" class="team-logo mb-1" :alt="game.team1" />
                        <span class="text-caption font-weight-bold">{{ game.score1 }}</span>
                    </div>
                    <div class="mx-1 opacity-50">
                        <v-icon size="16">mdi-minus</v-icon>
                    </div>
                    <div class="d-flex flex-column align-center mx-2">
                        <img :src="game.logo2" height="24" width="24" class="team-logo mb-1" :alt="game.team2" />
                        <span class="text-caption font-weight-bold">{{ game.score2 }}</span>
                    </div>
                </div>
                <div class="text-caption opacity-60" style="font-size: 0.65rem !important; line-height: 1;">
                    {{ game.team1 }} vs {{ game.team2 }}
                </div>
            </div>
        </div>

        <!-- Tall Mode: Single Game -->
        <div v-else class="d-flex flex-column align-center justify-center text-center w-100">
            <div class="d-flex align-center justify-center gap-6 mb-3">
                 <img :src="currentGame.logo1" height="42" width="42" class="team-logo" :alt="currentGame.team1" />
                 <span class="text-subtitle-1 font-weight-light opacity-50">vs</span>
                 <img :src="currentGame.logo2" height="42" width="42" class="team-logo" :alt="currentGame.team2" />
            </div>
            <div class="text-center">
                <div class="text-subtitle-1 font-weight-medium adaptive-text">{{ currentGame.league }}</div>
                <div class="text-caption adaptive-text opacity-60">{{ currentGame.game }}</div>
            </div>
            <div class="text-h6 adaptive-text font-weight-bold mt-1">{{ currentGame.score }}</div>
        </div>
    </div>
</template>

<script setup>
import { computed, ref, onMounted, onUnmounted } from 'vue';

const sportsData = [
    { league: 'NBA', game: 'Lakers vs Warriors', score: '102-98', team1: 'LAL', team2: 'GSW', score1: '102', score2: '98', logo1: 'https://a.espncdn.com/i/teamlogos/nba/500/lal.png', logo2: 'https://a.espncdn.com/i/teamlogos/nba/500/gsw.png' },
    { league: 'NFL', game: 'Chiefs vs Bills', score: '24-21', team1: 'KC', team2: 'BUF', score1: '24', score2: '21', logo1: 'https://a.espncdn.com/i/teamlogos/nfl/500/kc.png', logo2: 'https://a.espncdn.com/i/teamlogos/nfl/500/buf.png' },
    { league: 'MLB', game: 'Yankees vs Red Sox', score: '5-3', team1: 'NYY', team2: 'BOS', score1: '5', score2: '3', logo1: 'https://a.espncdn.com/i/teamlogos/mlb/500/nyy.png', logo2: 'https://a.espncdn.com/i/teamlogos/mlb/500/bos.png' },
    { league: 'NHL', game: 'Bruins vs Leafs', score: '4-2', team1: 'BOS', team2: 'TOR', score1: '4', score2: '2', logo1: 'https://a.espncdn.com/i/teamlogos/nhl/500/bos.png', logo2: 'https://a.espncdn.com/i/teamlogos/nhl/500/tor.png' },
    { league: 'NBA', game: 'Suns vs Mavs', score: '115-110', team1: 'PHX', team2: 'DAL', score1: '115', score2: '110', logo1: 'https://a.espncdn.com/i/teamlogos/nba/500/phx.png', logo2: 'https://a.espncdn.com/i/teamlogos/nba/500/dal.png' },
    { league: 'NFL', game: '49ers vs Lions', score: '34-31', team1: 'SF', team2: 'DET', score1: '34', score2: '31', logo1: 'https://a.espncdn.com/i/teamlogos/nfl/500/sf.png', logo2: 'https://a.espncdn.com/i/teamlogos/nfl/500/det.png' }
];

const props = defineProps({
    tall: Boolean,
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
.team-logo {
    object-fit: contain;
}
</style>
