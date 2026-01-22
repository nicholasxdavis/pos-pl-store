<template>
  <div class="fill-height pa-8 d-flex flex-column fade-in-up">
    <!-- Header -->
    <div class="d-flex align-center justify-space-between mb-6">
      <div class="d-flex align-center">
        <v-btn icon="mdi-arrow-left" variant="text" class="mr-2" @click="$router.back()"></v-btn>
        <h2 class="text-h4 font-weight-light adaptive-text">Sports Plugin</h2>
      </div>
      <div class="d-flex gap-2">
        <v-btn icon="mdi-refresh" variant="text" @click="refreshData"></v-btn>
      </div>
    </div>

    <!-- Sports Tabs -->
    <div class="mb-6 d-flex overflow-x-auto custom-scrollbar gap-2" :class="{'opacity-50': loading}">
      <v-chip
        v-for="sport in sportsList"
        :key="sport.id"
        :color="currentSport === sport.id ? themeColor : 'default'"
        :variant="currentSport === sport.id ? 'flat' : 'outlined'"
        class="text-uppercase"
        style="min-width: 80px; justify-content: center;"
        @click="currentSport = sport.id"
      >
        {{ sport.label }}
      </v-chip>
    </div>

    <!-- Loading State -->
    <div v-if="loading" class="d-flex justify-center align-center flex-grow-1">
      <v-progress-circular indeterminate :color="themeColor" size="64"></v-progress-circular>
    </div>

    <!-- Empty State -->
    <div v-else-if="filteredGames.length === 0 && currentSport !== 'news'" class="d-flex flex-column justify-center align-center flex-grow-1 opacity-60">
      <v-icon size="64" class="mb-4">mdi-stadium</v-icon>
      <div class="text-h6">No games scheduled</div>
      <div class="text-caption">Check back later or try another sport</div>
    </div>

    <!-- News View -->
    <div v-else-if="currentSport === 'news'" class="overflow-y-auto custom-scrollbar flex-grow-1">
       <v-row>
        <v-col v-for="news in newsData" :key="news.id" cols="12" md="6">
          <div class="glass-panel pa-0 h-100 d-flex flex-column cursor-pointer hover-scale overflow-hidden" @click="openLink(news.link)">
             <v-img
              v-if="news.image"
              :src="news.image"
              cover
              height="200"
              class="bg-grey-darken-3 w-100"
            ></v-img>
            <div class="d-flex flex-column pa-4 flex-grow-1">
              <div class="text-caption text-primary mb-1 font-weight-bold">{{ news.sport }}</div>
              <div class="text-subtitle-1 font-weight-medium mb-1 line-clamp-2" style="line-height: 1.2;">{{ news.headline }}</div>
              <div class="text-caption opacity-70 line-clamp-2 mb-2">{{ news.description }}</div>
              <div class="mt-auto text-caption opacity-50">{{ news.published }}</div>
            </div>
          </div>
        </v-col>
       </v-row>
    </div>

    <!-- Games List -->
    <div v-else class="overflow-y-auto custom-scrollbar flex-grow-1">
      <div class="glass-panel pa-0 overflow-hidden">
         <div 
            v-for="(game, index) in filteredGames" 
            :key="game.id" 
            class="d-flex align-center px-4 py-3 cursor-pointer hover-bg"
            :class="{ 'border-bottom': index < filteredGames.length - 1 }"
            @click="showGameDetail(game)"
         >
            <!-- Status (Left) -->
            <div class="d-flex flex-column mr-4" style="width: 60px;">
                <span :class="getStatusColor(game.status)">
                   <v-icon size="x-small" v-if="game.status === 'LIVE'" class="mr-1 flashing">mdi-circle-medium</v-icon>
                   {{ game.status === 'LIVE' ? 'LIVE' : game.status.slice(0, 5) }}
                </span>
                <span class="text-caption opacity-50">{{ game.statusDetail }}</span>
            </div>

            <!-- Teams -->
            <div class="flex-grow-1 d-flex flex-column justify-center" style="min-width: 0">
                 <!-- Away -->
                 <div class="d-flex align-center mb-1">
                     <v-avatar size="24" class="bg-grey-darken-3 mr-2">
                       <v-img :src="game.awayLogo" :lazy-src="fallbackLogo"></v-img>
                     </v-avatar>
                     <span class="font-weight-medium text-truncate">{{ game.awayTeam }}</span>
                     <v-spacer></v-spacer>
                     <span class="text-h6 font-weight-bold ml-4" :class="{'text-primary': parseInt(game.awayScore) > parseInt(game.homeScore)}">{{ game.awayScore }}</span>
                 </div>
                 <!-- Home -->
                 <div class="d-flex align-center">
                     <v-avatar size="24" class="bg-grey-darken-3 mr-2">
                       <v-img :src="game.homeLogo" :lazy-src="fallbackLogo"></v-img>
                     </v-avatar>
                     <span class="font-weight-medium text-truncate">{{ game.homeTeam }}</span>
                     <v-spacer></v-spacer>
                     <span class="text-h6 font-weight-bold ml-4" :class="{'text-primary': parseInt(game.homeScore) > parseInt(game.awayScore)}">{{ game.homeScore }}</span>
                 </div>
            </div>

            <!-- Chevron -->
             <v-icon class="ml-4 opacity-50">mdi-chevron-right</v-icon>
         </div>
      </div>
    </div>

    <!-- Detail Dialog -->
    <v-dialog v-model="dialog" max-width="500">
      <v-card v-if="selectedGame" class="pa-2">
        <v-card-title class="d-flex justify-space-between align-center">
             <span>{{ selectedGame.awayTeam }} @ {{ selectedGame.homeTeam }}</span>
        </v-card-title>
        
        <v-card-text class="pt-4">
            <!-- Big Score -->
             <div class="d-flex justify-center align-center gap-8 mb-6">
                  <div class="text-center">
                    <v-avatar size="64" class="mb-2 bg-grey-darken-3"><v-img :src="selectedGame.awayLogo"></v-img></v-avatar>
                    <div class="text-h4 font-weight-bold">{{ selectedGame.awayScore }}</div>
                  </div>
                   <div class="text-h5 opacity-30">VS</div>
                  <div class="text-center">
                    <v-avatar size="64" class="mb-2 bg-grey-darken-3"><v-img :src="selectedGame.homeLogo"></v-img></v-avatar>
                    <div class="text-h4 font-weight-bold">{{ selectedGame.homeScore }}</div>
                  </div>
             </div>

             <!-- Details List -->
             <v-list density="compact" class="bg-transparent">
                <v-list-item class="px-0">
                    <template v-slot:prepend><span class="opacity-60 text-caption mr-4" style="width: 60px">Status</span></template>
                    <v-list-item-title class="font-weight-bold">{{ selectedGame.status }}</v-list-item-title>
                </v-list-item>
                <v-divider class="opacity-10"></v-divider>
                <v-list-item class="px-0">
                    <template v-slot:prepend><span class="opacity-60 text-caption mr-4" style="width: 60px">Detail</span></template>
                    <v-list-item-title class="text-caption">{{ selectedGame.statusDetail }}</v-list-item-title>
                </v-list-item>
                <v-divider class="opacity-10" v-if="selectedGame.venue"></v-divider>
                <v-list-item class="px-0" v-if="selectedGame.venue">
                    <template v-slot:prepend><span class="opacity-60 text-caption mr-4" style="width: 60px">Venue</span></template>
                    <v-list-item-title class="text-caption">{{ selectedGame.venue }}</v-list-item-title>
                </v-list-item>
                <v-divider class="opacity-10" v-if="selectedGame.odds"></v-divider>
                 <v-list-item class="px-0" v-if="selectedGame.odds">
                    <template v-slot:prepend><span class="opacity-60 text-caption mr-4" style="width: 60px">Odds</span></template>
                    <v-list-item-title class="text-caption">{{ selectedGame.odds }}</v-list-item-title>
                </v-list-item>
             </v-list>
        </v-card-text>

        <v-card-actions>
          <span class="text-caption opacity-30 ml-4">Source: {{ selectedGame.source }}</span>
          <v-spacer></v-spacer>
          <v-btn variant="text" @click="dialog = false">Close</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted, watch } from 'vue';

// Props from PluginView.vue
const props = defineProps({
    themeColor: { type: String, default: '#6366f1' },
    isDarkTheme: { type: Boolean, default: true }
});

// Use prop for theme color
const themeColor = props.themeColor;

// State
const loading = ref(false);
const currentSport = ref('all');
const allGames = ref([]);
const newsData = ref([]);
const showingNews = ref(false);
const dialog = ref(false);
const selectedGame = ref(null);
const fallbackLogo = 'data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIyNCIgaGVpZ2h0PSIyNCIgdmlld0JveD0iMCAwIDI0IDI0Ij48cGF0aCBmaWxsPSIjNjY2IiBkPSJNMTIgMmEyIDEgMCAwIDEgMiAyYzAgMS4xLS45IDItMiAycy0yLS45LTItMiAuOS0yIDItMnptMCA0YTEwIDEwIDAgMSAwIDEwIDEwQTEwIDEwIDAgMCAwIDEyIDZ6bTAgMThhOCA4IDAgMSAxIDgtOCA4IDggMCAwIDEgLTggOHoiLz48L3N2Zz4='; // Simple SVG placeholder

const sportsList = [
  { id: 'all', label: 'All' },
  { id: 'news', label: 'News' },
  { id: 'nfl', label: 'NFL' },
  { id: 'nba', label: 'NBA' },
  { id: 'mlb', label: 'MLB' },
  { id: 'nhl', label: 'NHL' },
  { id: 'soccer', label: 'Soccer' },
  { id: 'ncaaf', label: 'NCAA FB' },
  { id: 'ncaab', label: 'NCAA BB' }
];

const filteredGames = computed(() => {
    if (currentSport.value === 'all') return allGames.value;
    if (currentSport.value === 'news') return []; // Handled by v-else-if
    return allGames.value.filter(g => g.sport.toLowerCase() === currentSport.value);
});

// Watch strict sport change to load news if selected
watch(currentSport, (newVal) => {
    if (newVal === 'news' && newsData.value.length === 0) {
        loadNews();
    }
});

// ESPN API Logic
const ESPN_BASE = 'https://site.api.espn.com/apis/site/v2/sports';
const ESPN_SPORTS = {
    nfl: 'football/nfl',
    nba: 'basketball/nba',
    mlb: 'baseball/mlb',
    nhl: 'hockey/nhl',
    ncaaf: 'football/college-football',
    ncaab: 'basketball/mens-college-basketball',
    soccer: 'soccer/usa.1'
};

async function fetchESPN(sport) {
    try {
        const path = ESPN_SPORTS[sport];
        if (!path) return [];
        const res = await fetch(`${ESPN_BASE}/${path}/scoreboard`);
        const data = await res.json();
        return data.events?.map(e => {
            const comp = e.competitions[0];
            const home = comp.competitors.find(t => t.homeAway === 'home');
            const away = comp.competitors.find(t => t.homeAway === 'away');
            return {
                id: e.id,
                sport: sport.toUpperCase(),
                status: e.status.type.state === 'in' ? 'LIVE' : e.status.type.completed ? 'FINAL' : 'UPCOMING',
                statusDetail: e.status.type.shortDetail,
                homeTeam: home.team.displayName,
                awayTeam: away.team.displayName,
                homeLogo: home.team.logo,
                awayLogo: away.team.logo,
                homeScore: home.score || '0',
                awayScore: away.score || '0',
                homeRecord: home.records?.[0]?.summary || '',
                awayRecord: away.records?.[0]?.summary || '',
                venue: comp.venue?.fullName || '',
                broadcast: comp.broadcasts?.[0]?.names?.[0] || '',
                odds: comp.odds?.[0]?.details || '',
                source: 'ESPN'
            };
        }) || [];
    } catch (e) { return []; }
}

async function fetchESPNNews(sport) {
     try {
        const path = ESPN_SPORTS[sport];
        if (!path) return [];
        const res = await fetch(`${ESPN_BASE}/${path}/news`);
        const data = await res.json();
        return data.articles?.slice(0, 5).map(a => ({
            id: a.id,
            sport: sport.toUpperCase(),
            headline: a.headline,
            description: a.description,
            image: a.images?.[0]?.url || '',
            link: a.links?.web?.href || '',
            published: new Date(a.published).toLocaleDateString()
        })) || [];
    } catch (e) { return []; }
}

async function refreshData() {
    loading.value = true;
    allGames.value = [];
    
    // Parallel fetch for games
    const promises = Object.keys(ESPN_SPORTS).map(s => fetchESPN(s));
    
    // Also fetch BallDontLie for NBA backup if needed (Checking config key)
    // For brevity/simplicity, using mainly ESPN as it's robust and free-ish public structure
    
    const results = await Promise.all(promises);
    const flattened = results.flat().sort((a,b) => {
        // Sort LIVE first
        if (a.status === 'LIVE' && b.status !== 'LIVE') return -1;
        if (a.status !== 'LIVE' && b.status === 'LIVE') return 1;
        return 0;
    });
    
    allGames.value = flattened;
    loading.value = false;
}

async function loadNews() {
    loading.value = true;
    const promises = Object.keys(ESPN_SPORTS).map(s => fetchESPNNews(s));
    const results = await Promise.all(promises);
    newsData.value = results.flat();
    loading.value = false;
}

function toggleNews() {
    showingNews.value = !showingNews.value;
    if (showingNews.value && newsData.value.length === 0) loadNews();
}

function showGameDetail(game) {
    selectedGame.value = game;
    dialog.value = true;
}

function openLink(url) {
    if(url) window.open(url, '_blank');
}

function getStatusColor(status) {
    if (status === 'LIVE') return 'text-error font-weight-bold';
    if (status === 'FINAL') return 'opacity-60';
    return 'text-success';
}

function getSportColor(sport) {
    // Simple mapping
    return 'white';
}

// Auto refresh
let interval;
onMounted(() => {
    refreshData();
    interval = setInterval(refreshData, 60000);
});
onUnmounted(() => clearInterval(interval));

</script>

<style scoped>
.d-grid-scoreboard {
    display: grid;
    grid-template-columns: 1fr auto 1fr;
}
.line-clamp-2 {
    display: -webkit-box;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
    overflow: hidden;
}
.flashing {
    animation: flash 2s infinite;
}
@keyframes flash {
    0% { opacity: 1; }
    50% { opacity: 0.5; }
    100% { opacity: 1; }
}
.hover-bg {
    transition: background-color 0.2s ease;
}
.hover-bg:hover {
    background: rgba(255, 255, 255, 0.05);
}
.gap-2 {
    gap: 8px;
}
.border-bottom {
    border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}
.bg-white-5 { background: rgba(255,255,255,0.05); }
.bg-black-50 { background: rgba(0,0,0,0.3); }
</style>
