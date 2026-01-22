<template>
  <div class="fill-height d-flex flex-column fade-in-up">
    
    <div v-if="activeNote" class="d-flex flex-column fill-height bg-paper">
        <div class="d-flex align-center justify-space-between px-4 py-3 border-bottom">
            <div class="d-flex align-center text-primary cursor-pointer" @click="saveAndClose">
                <v-icon icon="mdi-chevron-left" start></v-icon>
                <span class="text-body-1">Notes</span>
            </div>
            <div class="d-flex gap-2">
                 <v-btn icon="mdi-share-variant" variant="text" size="small" color="primary"></v-btn>
                 <v-btn icon="mdi-trash-can-outline" variant="text" size="small" color="error" @click="deleteActiveNote"></v-btn>
            </div>
        </div>

        <div class="d-flex flex-column flex-grow-1 px-6 py-4 overflow-y-auto custom-scrollbar">
            <input 
                v-model="activeNote.title" 
                class="text-h4 font-weight-bold mb-4 transparent-input title-input adaptive-text"
                placeholder="Title"
            />
            <div class="text-caption opacity-50 mb-4">{{ formatDate(activeNote.date) }}</div>
            <textarea 
                v-model="activeNote.body" 
                class="flex-grow-1 transparent-input body-input adaptive-text"
                placeholder="Start typing..."
            ></textarea>
        </div>
    </div>

    <div v-else class="d-flex flex-column fill-height pa-6">
        <div class="d-flex align-center justify-space-between mb-4">
             <div class="d-flex align-center">
                <v-btn icon="mdi-arrow-left" variant="text" class="mr-2" @click="$router.back()"></v-btn>
                <h2 class="text-h3 font-weight-bold adaptive-text">Notes</h2>
            </div>
            <v-btn icon="mdi-plus" color="primary" variant="tonal" size="large" @click="createNew"></v-btn>
        </div>

        <div class="mb-6">
            <div class="search-bar px-3 py-2 d-flex align-center">
                <v-icon icon="mdi-magnify" class="opacity-50 mr-2"></v-icon>
                <input 
                    v-model="searchQuery" 
                    placeholder="Search" 
                    class="flex-grow-1 transparent-input adaptive-text"
                />
            </div>
        </div>

        <div class="flex-grow-1 overflow-y-auto custom-scrollbar glass-panel pa-0 overflow-hidden">
             <div v-if="filteredNotes.length === 0" class="d-flex flex-column align-center justify-center h-100 opacity-50">
                 <v-icon size="64" class="mb-2">mdi-note-text-outline</v-icon>
                 <div>No notes</div>
             </div>

             <div 
                v-else
                v-for="(note, index) in filteredNotes" 
                :key="note.id"
                class="note-item px-4 py-3 cursor-pointer hover-bg"
                :class="{ 'border-bottom': index < filteredNotes.length - 1 }"
                @click="openNote(note)"
            >
                <div class="font-weight-bold mb-1 text-truncate">{{ note.title || 'New Note' }}</div>
                <div class="d-flex align-center text-caption opacity-60">
                    <span class="mr-2 font-weight-medium">{{ formatShortDate(note.date) }}</span>
                    <span class="text-truncate">{{ note.body || 'No additional text' }}</span>
                </div>
            </div>
        </div>
        
        <div class="text-center mt-4 text-caption opacity-50">
            {{ notes.length }} Notes
        </div>
    </div>

  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue';

const props = defineProps({
    themeColor: { type: String, default: '#F1C40F' }, // Default Yellow for Notes
    isDarkTheme: { type: Boolean, default: true }
});

// Mock Data / State
const notes = ref([
    { id: 1, title: 'Ideas for Project', body: '1. Clean UI\n2. Fast interactions\n3. Glassmorphism', date: new Date().toISOString() },
    { id: 2, title: 'Grocery List', body: 'Milk, Eggs, Bread, Coffee', date: new Date(Date.now() - 86400000).toISOString() },
    { id: 3, title: 'Meeting Notes', body: 'Discuss Q1 roadmap and hiring plans.', date: new Date(Date.now() - 172800000).toISOString() }
]);

const searchQuery = ref('');
const activeNote = ref(null);

// Computed
const filteredNotes = computed(() => {
    if (!searchQuery.value) return notes.value;
    const q = searchQuery.value.toLowerCase();
    return notes.value.filter(n => 
        (n.title && n.title.toLowerCase().includes(q)) || 
        (n.body && n.body.toLowerCase().includes(q))
    );
});

// Actions
function createNew() {
    const newNote = {
        id: Date.now(),
        title: '',
        body: '',
        date: new Date().toISOString()
    };
    notes.value.unshift(newNote); // Add to top
    activeNote.value = newNote;
}

function openNote(note) {
    activeNote.value = note; // pass by reference is fine for this mock
}

function saveAndClose() {
    // In a real app, save to storage here
    if (activeNote.value) {
        // Remove if empty
        if (!activeNote.value.title.trim() && !activeNote.value.body.trim()) {
            deleteActiveNote();
        } else {
            activeNote.value.date = new Date().toISOString(); // Update modified time
        }
    }
    activeNote.value = null;
}

function deleteActiveNote() {
    if (activeNote.value) {
        notes.value = notes.value.filter(n => n.id !== activeNote.value.id);
        activeNote.value = null;
    }
}

// Helpers
function formatDate(iso) {
    return new Date(iso).toLocaleString('en-US', { 
        month: 'long', day: 'numeric', year: 'numeric', hour: 'numeric', minute: '2-digit' 
    });
}

function formatShortDate(iso) {
    const d = new Date(iso);
    const now = new Date();
    if (d.toDateString() === now.toDateString()) {
        return d.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
    }
    return d.toLocaleDateString([], { month: 'numeric', day: 'numeric' });
}

</script>

<style scoped>
.gap-2 { gap: 8px; }

/* Custom Inputs to look like standard text */
.transparent-input {
    background: transparent;
    border: none;
    outline: none;
    color: inherit;
    width: 100%;
    font-family: inherit;
}

.title-input {
    line-height: 1.2;
}

.body-input {
    resize: none;
    line-height: 1.5;
    font-size: 1.1rem;
}

.search-bar {
    background: rgba(128, 128, 128, 0.15);
    border-radius: 10px;
}

/* List Styling */
.hover-bg { transition: background-color 0.2s ease; }
.hover-bg:hover { background: rgba(255, 255, 255, 0.05); }
.border-bottom { border-bottom: 1px solid rgba(128, 128, 128, 0.2); }

.glass-panel {
    border-radius: 16px;
    background: rgba(255, 255, 255, 0.05); /* Very subtle glass */
    backdrop-filter: blur(10px);
    -webkit-backdrop-filter: blur(10px);
    border: 1px solid rgba(255, 255, 255, 0.1);
}

.custom-scrollbar::-webkit-scrollbar {
    width: 4px;
}
.custom-scrollbar::-webkit-scrollbar-track {
    background: transparent;
}
.custom-scrollbar::-webkit-scrollbar-thumb {
    background: rgba(128, 128, 128, 0.3);
    border-radius: 2px;
}
</style>