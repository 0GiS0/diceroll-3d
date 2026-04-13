<script setup lang="ts">
import { ref, computed } from 'vue'

// State
const diceCount = ref(2)
const isRolling = ref(false)
const diceResults = ref<number[]>([])
const rollHistory = ref<{ dice: number[], total: number, timestamp: Date }[]>([])
const isDarkMode = ref(window.matchMedia('(prefers-color-scheme: dark)').matches)

// Animation states for each dice
const diceAnimations = ref<{ rotateX: number, rotateY: number, rotateZ: number }[]>([])

// Computed
const totalResult = computed(() => diceResults.value.reduce((a, b) => a + b, 0))
const hasResults = computed(() => diceResults.value.length > 0)

// Pip patterns for each dice face
const pipPatterns: Record<number, number[][]> = {
  1: [[1, 1]],
  2: [[0, 0], [2, 2]],
  3: [[0, 0], [1, 1], [2, 2]],
  4: [[0, 0], [0, 2], [2, 0], [2, 2]],
  5: [[0, 0], [0, 2], [1, 1], [2, 0], [2, 2]],
  6: [[0, 0], [0, 2], [1, 0], [1, 2], [2, 0], [2, 2]]
}

function getRotationForValue(value: number): { x: number, y: number } {
  const rotations: Record<number, { x: number, y: number }> = {
    1: { x: 0, y: 0 },
    2: { x: 0, y: -90 },
    3: { x: -90, y: 0 },
    4: { x: 90, y: 0 },
    5: { x: 0, y: 90 },
    6: { x: 180, y: 0 }
  }
  return rotations[value]
}

async function rollDice() {
  if (isRolling.value) return
  
  isRolling.value = true
  diceResults.value = []
  
  // Generate results
  const results: number[] = []
  for (let i = 0; i < diceCount.value; i++) {
    results.push(Math.floor(Math.random() * 6) + 1)
  }
  
  // Initialize animations with random starting rotations
  diceAnimations.value = results.map(() => ({
    rotateX: Math.random() * 720 - 360,
    rotateY: Math.random() * 720 - 360,
    rotateZ: Math.random() * 360 - 180
  }))
  
  // Animate through multiple random rotations
  const animationDuration = 1200
  const steps = 15
  const stepDuration = animationDuration / steps
  
  for (let step = 0; step < steps; step++) {
    await new Promise(resolve => setTimeout(resolve, stepDuration))
    
    if (step < steps - 1) {
      // Random intermediate rotations
      diceAnimations.value = diceAnimations.value.map(() => ({
        rotateX: Math.random() * 720 + 360,
        rotateY: Math.random() * 720 + 360,
        rotateZ: Math.random() * 360
      }))
    } else {
      // Final rotation to show result
      diceAnimations.value = results.map(result => {
        const final = getRotationForValue(result)
        return {
          rotateX: final.x + 360 * 2,
          rotateY: final.y + 360 * 2,
          rotateZ: 0
        }
      })
    }
  }
  
  // Final settle animation
  await new Promise(resolve => setTimeout(resolve, 300))
  diceAnimations.value = results.map(result => {
    const final = getRotationForValue(result)
    return { rotateX: final.x, rotateY: final.y, rotateZ: 0 }
  })
  
  diceResults.value = results
  rollHistory.value.unshift({
    dice: [...results],
    total: results.reduce((a, b) => a + b, 0),
    timestamp: new Date()
  })
  
  isRolling.value = false
}

function toggleDarkMode() {
  isDarkMode.value = !isDarkMode.value
}
</script>

<template>
  <div class="min-h-screen transition-colors duration-300" :class="isDarkMode ? 'bg-slate-900 text-white' : 'bg-gray-50 text-gray-900'">
    <!-- Header -->
    <header class="px-6 py-4 flex justify-between items-center">
      <h1 class="text-2xl font-bold tracking-tight">
        🎲 DiceRoll 3D
      </h1>
      <button 
        @click="toggleDarkMode"
        class="p-2 rounded-full transition-colors"
        :class="isDarkMode ? 'bg-slate-800 hover:bg-slate-700' : 'bg-gray-200 hover:bg-gray-300'"
        aria-label="Toggle dark mode"
      >
        <span v-if="isDarkMode" class="text-xl">☀️</span>
        <span v-else class="text-xl">🌙</span>
      </button>
    </header>

    <main class="container mx-auto px-4 pb-8">
      <div class="grid lg:grid-cols-3 gap-6">
        <!-- 3D Dice Area -->
        <div class="lg:col-span-2">
          <div 
            class="rounded-2xl overflow-hidden shadow-lg h-[350px] flex items-center justify-center perspective-1000"
            :class="isDarkMode ? 'bg-slate-800' : 'bg-gradient-to-br from-indigo-100 to-purple-100'"
            style="perspective: 1000px;"
          >
            <div class="flex gap-6 flex-wrap justify-center items-center p-4">
              <div 
                v-for="(anim, index) in diceAnimations" 
                :key="index"
                class="w-20 h-20 relative"
                style="transform-style: preserve-3d;"
              >
                <div 
                  class="w-full h-full transition-transform"
                  :style="{
                    transform: `rotateX(${anim.rotateX}deg) rotateY(${anim.rotateY}deg) rotateZ(${anim.rotateZ}deg)`,
                    transformStyle: 'preserve-3d',
                    transitionDuration: isRolling ? '80ms' : '300ms'
                  }"
                >
                  <!-- Face 1 (front) -->
                  <div class="dice-face" :class="isDarkMode ? 'bg-white' : 'bg-white'" style="transform: translateZ(40px);">
                    <div class="pip-container">
                      <span v-for="(pip, i) in pipPatterns[1]" :key="i" class="pip" :style="{ gridRow: pip[0]+1, gridColumn: pip[1]+1 }"></span>
                    </div>
                  </div>
                  <!-- Face 6 (back) -->
                  <div class="dice-face" :class="isDarkMode ? 'bg-white' : 'bg-white'" style="transform: rotateY(180deg) translateZ(40px);">
                    <div class="pip-container">
                      <span v-for="(pip, i) in pipPatterns[6]" :key="i" class="pip" :style="{ gridRow: pip[0]+1, gridColumn: pip[1]+1 }"></span>
                    </div>
                  </div>
                  <!-- Face 2 (right) -->
                  <div class="dice-face" :class="isDarkMode ? 'bg-white' : 'bg-white'" style="transform: rotateY(90deg) translateZ(40px);">
                    <div class="pip-container">
                      <span v-for="(pip, i) in pipPatterns[2]" :key="i" class="pip" :style="{ gridRow: pip[0]+1, gridColumn: pip[1]+1 }"></span>
                    </div>
                  </div>
                  <!-- Face 5 (left) -->
                  <div class="dice-face" :class="isDarkMode ? 'bg-white' : 'bg-white'" style="transform: rotateY(-90deg) translateZ(40px);">
                    <div class="pip-container">
                      <span v-for="(pip, i) in pipPatterns[5]" :key="i" class="pip" :style="{ gridRow: pip[0]+1, gridColumn: pip[1]+1 }"></span>
                    </div>
                  </div>
                  <!-- Face 3 (top) -->
                  <div class="dice-face" :class="isDarkMode ? 'bg-white' : 'bg-white'" style="transform: rotateX(90deg) translateZ(40px);">
                    <div class="pip-container">
                      <span v-for="(pip, i) in pipPatterns[3]" :key="i" class="pip" :style="{ gridRow: pip[0]+1, gridColumn: pip[1]+1 }"></span>
                    </div>
                  </div>
                  <!-- Face 4 (bottom) -->
                  <div class="dice-face" :class="isDarkMode ? 'bg-white' : 'bg-white'" style="transform: rotateX(-90deg) translateZ(40px);">
                    <div class="pip-container">
                      <span v-for="(pip, i) in pipPatterns[4]" :key="i" class="pip" :style="{ gridRow: pip[0]+1, gridColumn: pip[1]+1 }"></span>
                    </div>
                  </div>
                </div>
              </div>
              
              <!-- Placeholder when no dice -->
              <div v-if="diceAnimations.length === 0" class="text-center opacity-50">
                <p class="text-4xl mb-2">🎲</p>
                <p>Selecciona la cantidad de dados y pulsa "Lanzar"</p>
              </div>
            </div>
          </div>

          <!-- Controls -->
          <div 
            class="mt-4 p-6 rounded-2xl shadow-lg"
            :class="isDarkMode ? 'bg-slate-800' : 'bg-white'"
          >
            <!-- Dice Count Selector -->
            <div class="mb-6">
              <label class="block text-sm font-medium mb-3 opacity-70">Cantidad de dados</label>
              <div class="flex gap-2 flex-wrap">
                <button
                  v-for="n in 6"
                  :key="n"
                  @click="diceCount = n"
                  class="w-12 h-12 rounded-xl font-bold text-lg transition-all duration-200"
                  :class="[
                    diceCount === n 
                      ? 'bg-indigo-500 text-white shadow-lg scale-110' 
                      : isDarkMode 
                        ? 'bg-slate-700 hover:bg-slate-600' 
                        : 'bg-gray-100 hover:bg-gray-200'
                  ]"
                >
                  {{ n }}
                </button>
              </div>
            </div>

            <!-- Roll Button -->
            <button
              @click="rollDice"
              :disabled="isRolling"
              class="w-full py-4 rounded-xl font-bold text-lg transition-all bg-gradient-to-r from-indigo-500 to-purple-600 text-white hover:from-indigo-600 hover:to-purple-700 disabled:opacity-50 disabled:cursor-not-allowed shadow-lg hover:shadow-xl active:scale-[0.98]"
            >
              <span v-if="isRolling" class="flex items-center justify-center gap-2">
                <svg class="animate-spin h-5 w-5" viewBox="0 0 24 24">
                  <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" fill="none"></circle>
                  <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                </svg>
                Lanzando...
              </span>
              <span v-else>🎲 ¡Lanzar dados!</span>
            </button>

            <!-- Results -->
            <div v-if="hasResults" class="mt-6 text-center animate-fade-in">
              <div class="flex justify-center gap-3 mb-3 flex-wrap">
                <span 
                  v-for="(result, i) in diceResults" 
                  :key="i"
                  class="w-14 h-14 rounded-xl flex items-center justify-center text-2xl font-bold shadow-md"
                  :class="isDarkMode ? 'bg-slate-700' : 'bg-gray-100'"
                >
                  {{ result }}
                </span>
              </div>
              <p class="text-3xl font-bold">
                Total: <span class="text-indigo-500">{{ totalResult }}</span>
              </p>
            </div>
          </div>
        </div>

        <!-- History Panel -->
        <div 
          class="rounded-2xl shadow-lg p-6 h-fit max-h-[600px] overflow-y-auto"
          :class="isDarkMode ? 'bg-slate-800' : 'bg-white'"
        >
          <h2 class="text-lg font-bold mb-4 flex items-center gap-2">
            📜 Historial
          </h2>
          
          <div v-if="rollHistory.length === 0" class="text-center py-8 opacity-50">
            <p>No hay tiradas aún</p>
            <p class="text-sm mt-1">¡Lanza los dados para comenzar!</p>
          </div>
          
          <div v-else class="space-y-3">
            <div 
              v-for="(roll, index) in rollHistory" 
              :key="index"
              class="p-3 rounded-xl transition-colors"
              :class="[
                isDarkMode ? 'bg-slate-700/50' : 'bg-gray-50',
                index === 0 ? 'ring-2 ring-indigo-500/30' : ''
              ]"
            >
              <div class="flex items-center justify-between gap-2">
                <div class="flex gap-1 flex-wrap">
                  <span 
                    v-for="(d, i) in roll.dice" 
                    :key="i"
                    class="w-8 h-8 rounded-lg flex items-center justify-center text-sm font-medium"
                    :class="isDarkMode ? 'bg-slate-600' : 'bg-white shadow-sm'"
                  >
                    {{ d }}
                  </span>
                </div>
                <span class="font-bold text-indigo-500 whitespace-nowrap">= {{ roll.total }}</span>
              </div>
              <p class="text-xs opacity-50 mt-1">
                {{ roll.timestamp.toLocaleTimeString() }}
              </p>
            </div>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<style>
.dice-face {
  position: absolute;
  width: 80px;
  height: 80px;
  border-radius: 12px;
  box-shadow: inset 0 0 20px rgba(0,0,0,0.1), 0 4px 8px rgba(0,0,0,0.1);
  backface-visibility: visible;
  display: flex;
  align-items: center;
  justify-content: center;
}

.pip-container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(3, 1fr);
  gap: 4px;
  width: 50px;
  height: 50px;
}

.pip {
  width: 12px;
  height: 12px;
  background: #1e1b4b;
  border-radius: 50%;
  justify-self: center;
  align-self: center;
}

@keyframes fade-in {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

.animate-fade-in {
  animation: fade-in 0.3s ease-out;
}
</style>
