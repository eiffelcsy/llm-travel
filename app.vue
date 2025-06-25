<template>
  <div class="min-h-screen bg-white flex items-center justify-center px-6">
    <div class="w-full max-w-2xl">
      <!-- Contextual Prompt -->
      <div class="text-center mb-8">
        <div class="text-sm text-gray-400 mb-2">
          {{ timeContext }}
        </div>
        <h1 class="text-2xl font-light text-gray-600">
          {{ contextualPrompt }}
        </h1>
      </div>

      <!-- Natural Input -->
      <div class="relative">
        <div class="flex items-center">
          <span class="text-lg font-light text-gray-500 mr-3">I want to</span>
          <div class="flex-1 relative">
            <input
              v-model="travelInput"
              @keydown.enter="handleSubmit"
              :placeholder="inputPlaceholder"
              class="w-full text-lg font-light text-gray-700 bg-transparent border-none outline-none placeholder-gray-300"
              ref="inputField"
              autocomplete="off"
            />
            <div class="absolute bottom-0 left-0 right-0 h-px bg-gradient-to-r from-transparent via-gray-200 to-transparent"></div>
          </div>
        </div>
      </div>

      <!-- Subtle Submit Hint -->
      <div class="text-center mt-6">
        <div class="text-xs text-gray-300 opacity-0 transition-opacity duration-300" :class="{ 'opacity-100': travelInput.length > 3 }">
          Press Enter to begin your journey
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
const travelInput = ref('')
const inputField = ref(null)

// Contextual prompts based on time of day
const getContextualPrompt = () => {
  const hour = new Date().getHours()
  
  if (hour >= 5 && hour < 12) {
    return "Where will today take you?"
  } else if (hour >= 12 && hour < 17) {
    return "Ready for your next adventure?"
  } else if (hour >= 17 && hour < 21) {
    return "Time to plan something special?"
  } else {
    return "Dreaming of distant places?"
  }
}

const getTimeContext = () => {
  const hour = new Date().getHours()
  
  if (hour >= 5 && hour < 12) {
    return "Good morning"
  } else if (hour >= 12 && hour < 17) {
    return "Good afternoon"
  } else if (hour >= 17 && hour < 21) {
    return "Good evening"
  } else {
    return "Good evening"
  }
}

const getInputPlaceholder = () => {
  const placeholders = [
    "explore the hidden temples of Kyoto",
    "find the perfect beach in Croatia", 
    "discover local markets in Morocco",
    "plan a weekend in the mountains",
    "experience the Northern Lights",
    "taste authentic pasta in Italy"
  ]
  return placeholders[Math.floor(Math.random() * placeholders.length)]
}

const contextualPrompt = ref(getContextualPrompt())
const timeContext = ref(getTimeContext())
const inputPlaceholder = ref(getInputPlaceholder())

const handleSubmit = () => {
  if (!travelInput.value.trim()) return
  
  // Here you would handle the travel request
  console.log('Travel request:', travelInput.value)
  
  // For now, just clear the input
  travelInput.value = ''
}

// Update prompts every minute to keep them contextual
onMounted(() => {
  if (inputField.value) {
    inputField.value.focus()
  }
  
  setInterval(() => {
    contextualPrompt.value = getContextualPrompt()
    timeContext.value = getTimeContext()
  }, 60000)
})
</script>
