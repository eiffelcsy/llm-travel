<template>
  <div class="min-h-screen bg-gradient-to-br from-zinc-50 via-white to-zinc-100 dark:from-zinc-900 dark:via-zinc-800 dark:to-zinc-900 transition-all duration-700" :class="showItinerary ? 'pr-96' : ''">
    <div class="flex items-center justify-center min-h-screen px-6 py-12 transition-all duration-700" :class="showItinerary ? 'transform -translate-x-12' : ''">
      <div class="max-w-2xl w-full">
        <div class="text-center mb-8">
          <h1 class="text-4xl font-light text-zinc-800 dark:text-zinc-200">
            Hi, {{ user }}.
          </h1>
        </div>
        
        <div class="relative">
          <div 
            class="transition-all duration-300 ease-in-out transform hover:scale-[1.02] focus-within:scale-[1.02]"
            :class="isInputFocused ? 'opacity-100' : 'opacity-60 hover:opacity-80'"
          >
            <input
              ref="inputRef"
              v-model="query"
              @focus="isInputFocused = true"
              @blur="isInputFocused = false"
              @keydown.enter="handleQuery"
              placeholder="Where is your next destination?"
              class="w-full px-6 py-4 text-lg bg-white/70 dark:bg-zinc-800/70 backdrop-blur-sm border border-zinc-200/50 dark:border-zinc-700/50 rounded-2xl shadow-lg placeholder-zinc-400 dark:placeholder-zinc-500 text-zinc-800 dark:text-zinc-200 transition-all duration-300"
            />
          </div>
          
          <div 
            class="mt-4 text-center transition-all duration-300"
            :class="isInputFocused ? 'opacity-100' : 'opacity-0'"
          >
            <p class="text-sm text-zinc-500 dark:text-zinc-400 font-light">
              Press Enter to begin your journey
            </p>
          </div>
        </div>

        <!-- Quick Demo Buttons -->
        <div v-if="!hasDestination" class="mt-8 text-center">
          <p class="text-sm text-zinc-500 dark:text-zinc-400 mb-4">Quick demo:</p>
          <button
            @click="simulateConversation"
            class="px-4 py-2 bg-zinc-100 dark:bg-zinc-800 hover:bg-zinc-200 dark:hover:bg-zinc-700 text-zinc-700 dark:text-zinc-300 rounded-lg transition-colors text-sm font-medium"
          >
            Simulate "Paris trip for 3 days"
          </button>
        </div>
      </div>
    </div>
    
    <div class="absolute bottom-6 left-1/2 transform -translate-x-1/2">
      <div class="text-xs text-zinc-400 dark:text-zinc-600 font-light tracking-wider">
        Powered by Wanderlust
      </div>
    </div>

    <!-- Itinerary Panel -->
    <ItineraryPanel
      :destination="destination"
      :start-date="startDate"
      :end-date="endDate"
      :travelers="travelers"
      :visible="showItinerary"
    />
  </div>
</template>

<script lang="ts" setup>
import { ref } from 'vue'
import ItineraryPanel from '~/components/ItineraryPanel.vue'

// Reactive state
const query = ref('')
const isInputFocused = ref(false)
const inputRef = ref<HTMLInputElement>()
const user = ref('John') // Hardcoded for now, will be integrated with auth flow later

// Itinerary state
const destination = ref('')
const startDate = ref<Date>()
const endDate = ref<Date>()
const travelers = ref(1)
const showItinerary = ref(false)
const hasDestination = ref(false)

function handleQuery() {
  if (query.value.trim()) {
    // TODO: Implement LLM query handling logic
    console.log('Query submitted:', query.value)
    
    // Simple simulation - detect if query mentions destination
    const queryLower = query.value.toLowerCase()
    if (queryLower.includes('paris') || queryLower.includes('trip') || queryLower.includes('travel')) {
      simulateConversation()
    }
    
    query.value = ''
    inputRef.value?.blur()
  }
}

function simulateConversation() {
  // Simulate the conversation flow where destination and dates are confirmed
  destination.value = 'Paris, France'
  startDate.value = new Date('2024-03-15')
  endDate.value = new Date('2024-03-17')
  travelers.value = 1
  hasDestination.value = true
  
  // Show itinerary panel with a slight delay for better UX
  setTimeout(() => {
    showItinerary.value = true
  }, 500)
}

useHead({
  title: 'Explore - Your Travel Companion',
  meta: [
    { name: 'description', content: 'Discover your next adventure with intelligent travel guidance' }
  ]
})
</script>

<style scoped>
/* Subtle animations */
.fade-enter-active,
.fade-leave-active {
  transition: all 0.3s ease-in-out;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translateY(10px);
}

::-webkit-scrollbar {
  width: 6px;
}

::-webkit-scrollbar-track {
  @apply bg-transparent;
}

::-webkit-scrollbar-thumb {
  @apply bg-zinc-300 dark:bg-zinc-600 rounded-full;
}

::-webkit-scrollbar-thumb:hover {
  @apply bg-zinc-400 dark:bg-zinc-500;
}
</style>