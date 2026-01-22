<template>
    <div class="notepad-widget d-flex flex-column h-100 w-100 text-left">
        <div class="d-flex align-center justify-space-between mb-2">
            <div class="align-self-start">
                <div 
                    class="text-overline adaptive-text opacity-80 mb-1"
                    :style="{ color: isDarkTheme ? 'inherit' : '#333333' }"
                >NOTES</div>
            </div>
            <v-icon size="small" :color="themeColor">mdi-pencil-outline</v-icon>
        </div>

        <div class="flex-grow-1 overflow-hidden d-flex flex-column">
            
            <div v-if="size === 'wide'" class="d-flex flex-column gap-1 w-100">
                 <div 
                    v-for="(note, i) in recentNotes.slice(0, 3)" 
                    :key="i" 
                    class="d-flex align-center py-1 border-bottom-light"
                >
                    <div class="text-caption font-weight-bold text-truncate flex-grow-1 mr-2">{{ note.title || 'New Note' }}</div>
                    <div class="text-caption opacity-50" style="font-size: 0.65rem;">{{ formatTime(note.date) }}</div>
                 </div>
                 <div v-if="recentNotes.length === 0" class="text-caption opacity-50">No notes yet</div>
            </div>

            <div v-else class="d-flex flex-column h-100">
                <div v-if="recentNotes.length > 0">
                    <div class="text-subtitle-2 font-weight-bold line-clamp-2 mb-1" style="line-height: 1.2;">
                        {{ recentNotes[0].title || 'New Note' }}
                    </div>
                    <div class="text-caption opacity-60 line-clamp-3">
                        {{ recentNotes[0].body }}
                    </div>
                    <div class="mt-auto pt-2 text-caption opacity-40">
                        {{ formatTime(recentNotes[0].date) }}
                    </div>
                </div>
                <div v-else class="d-flex flex-column align-center justify-center h-100 opacity-50">
                    <span class="text-caption">Tap to create</span>
                </div>
            </div>

        </div>
    </div>
</template>

<script setup>
import { computed } from 'vue';

// Mock data shared with App.vue in a real app, 
// here repeated or passed via store if available.
const mockNotes = [
    { title: 'Ideas for Project', body: '1. Clean UI\n2. Fast interactions', date: new Date().toISOString() },
    { title: 'Grocery List', body: 'Milk, Eggs, Bread, Coffee', date: new Date(Date.now() - 86400000).toISOString() },
    { title: 'Meeting Notes', body: 'Discuss Q1 roadmap and hiring plans.', date: new Date(Date.now() - 172800000).toISOString() }
];

const props = defineProps({
    size: { type: String, default: 'square' },
    themeColor: { type: String, default: '#F1C40F' },
    isDarkTheme: Boolean
});

const recentNotes = computed(() => mockNotes);

function formatTime(iso) {
    const d = new Date(iso);
    // Return time if today, date otherwise
    const now = new Date();
    if (d.toDateString() === now.toDateString()) {
         return d.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
    }
    return d.toLocaleDateString([], { month: 'numeric', day: 'numeric' });
}
</script>

<style scoped>
.gap-1 { gap: 4px; }

.border-bottom-light {
    border-bottom: 1px solid rgba(128, 128, 128, 0.1);
}

.line-clamp-2 {
    display: -webkit-box;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
    overflow: hidden;
}

.line-clamp-3 {
    display: -webkit-box;
    -webkit-line-clamp: 3;
    -webkit-box-orient: vertical;
    overflow: hidden;
}
</style>