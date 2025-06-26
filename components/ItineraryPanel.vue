<template>
  <div class="relative">
    <!-- Toggle Button -->
    <button
      v-if="!isExpanded"
      @click="togglePanel"
      class="fixed right-6 top-1/2 transform -translate-y-1/2 z-30 bg-white dark:bg-zinc-800 border border-zinc-200 dark:border-zinc-700 rounded-l-2xl px-4 py-6 shadow-lg hover:shadow-xl transition-all duration-500"
    >
      <div class="flex flex-col items-center space-y-2">
        <svg class="w-5 h-5 text-zinc-600 dark:text-zinc-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5H7a2 2 0 00-2 2v10a2 2 0 002 2h8a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2" />
        </svg>
        <span class="text-xs text-zinc-600 dark:text-zinc-400 font-light writing-mode-vertical">Itinerary</span>
      </div>
    </button>

    <!-- Main Panel -->
    <div
      class="fixed right-0 top-0 h-full w-96 bg-white dark:bg-zinc-900 border-l border-zinc-200 dark:border-zinc-700 shadow-xl z-30 transform transition-all duration-700 ease-out"
      :class="isExpanded ? 'translate-x-0 opacity-100' : 'translate-x-full opacity-0'"
    >
      <!-- Header -->
      <div class="flex items-center justify-between p-6 border-b border-zinc-200 dark:border-zinc-700 animate-fade-in-down" style="animation-delay: 0.1s;">
        <div>
          <h2 class="text-xl font-light text-zinc-800 dark:text-zinc-200">Your Itinerary</h2>
          <p class="text-sm text-zinc-500 dark:text-zinc-400 mt-1">{{ destination }} • {{ formatDateRange() }}</p>
        </div>
        <button
          @click="togglePanel"
          class="p-2 hover:bg-zinc-100 dark:hover:bg-zinc-800 rounded-lg transition-colors duration-200"
        >
          <svg class="w-5 h-5 text-zinc-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
          </svg>
        </button>
      </div>

      <!-- Cost Summary -->
      <div class="p-4 bg-zinc-50 dark:bg-zinc-800/50 border-b border-zinc-200 dark:border-zinc-700 animate-fade-in-down" style="animation-delay: 0.2s;">
        <div class="flex justify-between items-center">
          <span class="text-sm text-zinc-600 dark:text-zinc-400">Total Cost</span>
          <span class="text-lg font-medium text-zinc-800 dark:text-zinc-200">${{ totalCost.toLocaleString() }}</span>
        </div>
        <div class="flex justify-between items-center mt-1">
          <span class="text-xs text-zinc-500 dark:text-zinc-500">Per person</span>
          <span class="text-sm text-zinc-600 dark:text-zinc-400">${{ Math.round(totalCost / travelers).toLocaleString() }}</span>
        </div>
      </div>

      <!-- Content Area -->
      <div class="flex-1 overflow-y-auto h-[calc(100vh-200px)]">
        <!-- Days Timeline -->
        <div class="p-4 space-y-6">
          <div
            v-for="(day, dayIndex) in itineraryDays"
            :key="dayIndex"
            class="relative animate-fade-in-up"
            :style="{ 'animation-delay': `${0.3 + dayIndex * 0.1}s` }"
          >
            <!-- Day Header -->
            <div class="flex items-center justify-between mb-4">
              <div>
                <h3 class="font-medium text-zinc-800 dark:text-zinc-200">
                  Day {{ dayIndex + 1 }}
                </h3>
                <p class="text-sm text-zinc-500 dark:text-zinc-400">
                  {{ formatDate(day.date) }}
                </p>
              </div>
              <button
                @click="addActivity(dayIndex)"
                class="p-1 hover:bg-zinc-100 dark:hover:bg-zinc-800 rounded-lg transition-colors"
              >
                <svg class="w-4 h-4 text-zinc-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4" />
                </svg>
              </button>
            </div>

            <!-- Timeline Line -->
            <div class="absolute left-4 top-12 bottom-0 w-px bg-zinc-200 dark:bg-zinc-700"></div>

            <!-- Activities -->
            <div class="space-y-3">
              <div
                v-for="(activity, activityIndex) in day.activities"
                :key="activity.id"
                :draggable="true"
                @dragstart="handleDragStart(dayIndex, activityIndex)"
                @dragover.prevent
                @drop="handleDrop(dayIndex, activityIndex)"
                class="relative group"
              >
                <!-- Timeline Dot -->
                <div class="absolute left-2 w-3 h-3 bg-white dark:bg-zinc-900 border-2 border-zinc-300 dark:border-zinc-600 rounded-full transform -translate-x-1/2"></div>

                <!-- Activity Card -->
                <div class="ml-8 p-3 bg-white dark:bg-zinc-800 border border-zinc-200 dark:border-zinc-700 rounded-lg shadow-sm hover:shadow-lg transition-all duration-300 cursor-move animate-fade-in-right" :style="{ 'animation-delay': `${0.4 + dayIndex * 0.1 + activityIndex * 0.05}s` }">
                  <div class="flex justify-between items-start">
                    <div class="flex-1">
                      <!-- Time -->
                      <div
                        v-if="activity.time"
                        @click="editActivityTime(dayIndex, activityIndex)"
                        class="text-xs text-zinc-500 dark:text-zinc-400 mb-1 cursor-pointer hover:text-zinc-700 dark:hover:text-zinc-300"
                      >
                        {{ activity.time }}
                      </div>

                      <!-- Title -->
                      <h4
                        @click="editActivityTitle(dayIndex, activityIndex)"
                        class="font-medium text-zinc-800 dark:text-zinc-200 cursor-pointer hover:text-zinc-600 dark:hover:text-zinc-400"
                      >
                        {{ activity.title }}
                      </h4>

                      <!-- Description -->
                      <p
                        v-if="activity.description"
                        @click="editActivityDescription(dayIndex, activityIndex)"
                        class="text-sm text-zinc-600 dark:text-zinc-400 mt-1 cursor-pointer hover:text-zinc-500 dark:hover:text-zinc-500"
                      >
                        {{ activity.description }}
                      </p>

                      <!-- Cost -->
                      <div
                        v-if="activity.cost"
                        @click="editActivityCost(dayIndex, activityIndex)"
                        class="text-sm text-zinc-700 dark:text-zinc-300 mt-2 cursor-pointer hover:text-zinc-600 dark:hover:text-zinc-400"
                      >
                        ${{ activity.cost }}
                      </div>
                    </div>

                    <!-- Actions -->
                    <div class="flex space-x-1 opacity-0 group-hover:opacity-100 transition-opacity">
                      <button
                        @click="removeActivity(dayIndex, activityIndex)"
                        class="p-1 hover:bg-red-100 dark:hover:bg-red-900/30 rounded transition-colors"
                      >
                        <svg class="w-3 h-3 text-red-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
                        </svg>
                      </button>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Footer Actions -->
      <div class="border-t border-zinc-200 dark:border-zinc-700 p-4 animate-fade-in-up" style="animation-delay: 0.8s;">
        <div class="flex space-x-2">
          <button
            @click="shareItinerary"
            class="flex-1 px-3 py-2 bg-zinc-100 dark:bg-zinc-800 hover:bg-zinc-200 dark:hover:bg-zinc-700 text-zinc-700 dark:text-zinc-300 rounded-lg transition-all duration-200 text-sm font-medium hover:scale-105"
          >
            Share
          </button>
          <button
            @click="exportPDF"
            class="flex-1 px-3 py-2 bg-zinc-800 dark:bg-zinc-200 hover:bg-zinc-700 dark:hover:bg-zinc-300 text-white dark:text-zinc-800 rounded-lg transition-all duration-200 text-sm font-medium hover:scale-105"
          >
            Export PDF
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, computed, watch } from 'vue'

interface Activity {
  id: string
  title: string
  description?: string
  time?: string
  cost?: number
  type: 'flight' | 'hotel' | 'activity' | 'meal' | 'transport'
}

interface ItineraryDay {
  date: Date
  activities: Activity[]
}

// Props
interface Props {
  destination?: string
  startDate?: Date
  endDate?: Date
  travelers?: number
  visible?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  destination: '',
  travelers: 1,
  visible: false
})

// Reactive state
const isExpanded = ref(props.visible)
const draggedItem = ref<{ dayIndex: number; activityIndex: number } | null>(null)

// Sample data - in real app this would come from the conversation context
const itineraryDays = ref<ItineraryDay[]>([
  {
    date: new Date('2024-03-15'),
    activities: [
      {
        id: '1',
        title: 'Flight to Paris',
        description: 'Departure from JFK Airport',
        time: '9:00 AM',
        cost: 650,
        type: 'flight'
      },
      {
        id: '2',
        title: 'Hotel Check-in',
        description: 'Le Marais Boutique Hotel',
        time: '3:00 PM',
        cost: 120,
        type: 'hotel'
      },
      {
        id: '3',
        title: 'Dinner at L\'As du Fallafel',
        description: 'Traditional French dining',
        time: '7:00 PM',
        cost: 45,
        type: 'meal'
      }
    ]
  },
  {
    date: new Date('2024-03-16'),
    activities: [
      {
        id: '4',
        title: 'Visit Louvre Museum',
        description: 'Skip-the-line tickets included',
        time: '10:00 AM',
        cost: 25,
        type: 'activity'
      },
      {
        id: '5',
        title: 'Seine River Cruise',
        description: '1-hour sightseeing cruise',
        time: '4:00 PM',
        cost: 15,
        type: 'activity'
      }
    ]
  }
])

// Computed properties
const totalCost = computed(() => {
  return itineraryDays.value.reduce((total, day) => {
    return total + day.activities.reduce((dayTotal, activity) => {
      return dayTotal + (activity.cost || 0)
    }, 0)
  }, 0)
})

// Watch for prop changes
watch(() => props.visible, (newVal) => {
  isExpanded.value = newVal
})

// Methods
function togglePanel() {
  isExpanded.value = !isExpanded.value
}

function formatDate(date: Date): string {
  return date.toLocaleDateString('en-US', { 
    weekday: 'long', 
    month: 'short', 
    day: 'numeric' 
  })
}

function formatDateRange(): string {
  if (!props.startDate || !props.endDate) return ''
  const start = props.startDate.toLocaleDateString('en-US', { month: 'short', day: 'numeric' })
  const end = props.endDate.toLocaleDateString('en-US', { month: 'short', day: 'numeric' })
  return `${start} - ${end}`
}

function addActivity(dayIndex: number) {
  const newActivity: Activity = {
    id: Date.now().toString(),
    title: 'New Activity',
    time: '12:00 PM',
    cost: 0,
    type: 'activity'
  }
  itineraryDays.value[dayIndex].activities.push(newActivity)
}

function removeActivity(dayIndex: number, activityIndex: number) {
  itineraryDays.value[dayIndex].activities.splice(activityIndex, 1)
}

function handleDragStart(dayIndex: number, activityIndex: number) {
  draggedItem.value = { dayIndex, activityIndex }
}

function handleDrop(dayIndex: number, activityIndex: number) {
  if (!draggedItem.value) return
  
  const draggedActivity = itineraryDays.value[draggedItem.value.dayIndex].activities[draggedItem.value.activityIndex]
  
  // Remove from original position
  itineraryDays.value[draggedItem.value.dayIndex].activities.splice(draggedItem.value.activityIndex, 1)
  
  // Insert at new position
  itineraryDays.value[dayIndex].activities.splice(activityIndex, 0, draggedActivity)
  
  draggedItem.value = null
}

// Inline editing functions (simplified - in real app would use proper input components)
function editActivityTitle(dayIndex: number, activityIndex: number) {
  const newTitle = prompt('Edit activity title:', itineraryDays.value[dayIndex].activities[activityIndex].title)
  if (newTitle) {
    itineraryDays.value[dayIndex].activities[activityIndex].title = newTitle
  }
}

function editActivityDescription(dayIndex: number, activityIndex: number) {
  const newDescription = prompt('Edit description:', itineraryDays.value[dayIndex].activities[activityIndex].description)
  if (newDescription !== null) {
    itineraryDays.value[dayIndex].activities[activityIndex].description = newDescription
  }
}

function editActivityTime(dayIndex: number, activityIndex: number) {
  const newTime = prompt('Edit time:', itineraryDays.value[dayIndex].activities[activityIndex].time)
  if (newTime) {
    itineraryDays.value[dayIndex].activities[activityIndex].time = newTime
  }
}

function editActivityCost(dayIndex: number, activityIndex: number) {
  const newCost = prompt('Edit cost:', itineraryDays.value[dayIndex].activities[activityIndex].cost?.toString())
  if (newCost) {
    itineraryDays.value[dayIndex].activities[activityIndex].cost = parseFloat(newCost) || 0
  }
}

function shareItinerary() {
  // Generate shareable link
  const shareData = {
    destination: props.destination,
    startDate: props.startDate,
    endDate: props.endDate,
    itinerary: itineraryDays.value
  }
  
  // In real app, this would generate a proper shareable link
  const shareableLink = `${window.location.origin}/shared/${btoa(JSON.stringify(shareData))}`
  navigator.clipboard.writeText(shareableLink)
  
  // Show feedback (you might want to add a toast notification here)
  console.log('Shareable link copied to clipboard:', shareableLink)
}

function exportPDF() {
  // In real app, this would generate and download a PDF
  console.log('Exporting PDF...', {
    destination: props.destination,
    itinerary: itineraryDays.value,
    totalCost: totalCost.value
  })
}

// Expose methods for parent component to trigger
defineExpose({
  togglePanel,
  addActivity,
  updateFromConversation: (data: any) => {
    // Method to update itinerary from conversation context
    console.log('Updating itinerary from conversation:', data)
  }
})
</script>

<style scoped>
.writing-mode-vertical {
  writing-mode: vertical-rl;
  text-orientation: mixed;
}

/* Smooth animations */
@keyframes fade-in-down {
  from {
    opacity: 0;
    transform: translateY(-20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes fade-in-up {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes fade-in-right {
  from {
    opacity: 0;
    transform: translateX(20px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

.animate-fade-in-down {
  animation: fade-in-down 0.6s ease-out forwards;
  opacity: 0;
}

.animate-fade-in-up {
  animation: fade-in-up 0.6s ease-out forwards;
  opacity: 0;
}

.animate-fade-in-right {
  animation: fade-in-right 0.6s ease-out forwards;
  opacity: 0;
}

/* Custom scrollbar */
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

/* Drag and drop styling */
.cursor-move:hover {
  cursor: grab;
}

.cursor-move:active {
  cursor: grabbing;
}

/* Enhanced hover effects */
.group:hover .group-hover\:scale-105 {
  transform: scale(1.05);
}
</style> 