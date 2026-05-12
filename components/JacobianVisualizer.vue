<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps<{
  clicks: number
}>()

// Step 0: Initial grid
// Step 1: Mutation highlight
// Step 2: Jacobian effect arrows
// Step 3: Heatmap result

const isStep1 = computed(() => props.clicks >= 1)
const isStep2 = computed(() => props.clicks >= 2)
const isStep3 = computed(() => props.clicks >= 3)

const sequence = "MKYLL".split("")
const aminoAcids = ["A", "R", "N", "D", "C"] // Simplified list for visualization

</script>

<template>
  <div class="flex flex-col items-center justify-center w-full bg-white rounded-xl p-8 shadow-md border border-gray-200 font-mono overflow-hidden">
    
    <div class="relative flex flex-col items-center">
      <!-- Input Sequence -->
      <div class="flex space-x-2 mb-12">
        <div v-for="(char, i) in sequence" :key="i" 
          class="w-10 h-10 flex items-center justify-center rounded border-2 transition-all duration-500"
          :class="[
            isStep1 && i === 2 ? 'border-red-500 bg-red-50 scale-110 shadow-lg' : 'border-gray-300 bg-gray-50'
          ]"
        >
          <span v-if="isStep1 && i === 2" class="text-red-600 font-bold">Y→A</span>
          <span v-else>{{ char }}</span>
        </div>
      </div>

      <!-- Arrow down to Model -->
      <div class="h-10 w-1 bg-gray-200 mb-2 relative">
         <div class="absolute bottom-0 left-1/2 -translate-x-1/2 w-3 h-3 border-r-2 border-b-2 border-gray-200 rotate-45"></div>
      </div>

      <!-- Black Box / ESM-2 -->
      <div class="w-48 h-16 bg-blue-600 rounded-lg flex items-center justify-center text-white font-bold shadow-lg mb-8 relative z-10">
        ESM-2 Model
        
        <!-- Jacobian Arrows Out -->
        <div v-if="isStep2" class="absolute inset-0 pointer-events-none">
            <div v-for="i in 5" :key="i" 
                class="absolute top-1/2 left-1/2 h-20 w-0.5 bg-red-400 origin-top transition-all duration-1000"
                :style="{ transform: `rotate(${(i-3)*40}deg) translate(0, 10px)` }"
            >
                <div class="absolute bottom-0 left-1/2 -translate-x-1/2 w-2 h-2 border-r-2 border-b-2 border-red-400 rotate-45"></div>
            </div>
        </div>
      </div>

      <!-- Output Heatmap / Jacobian -->
      <div class="mt-16 grid grid-cols-5 gap-1 transition-all duration-1000"
        :class="isStep3 ? 'opacity-100 translate-y-0' : 'opacity-0 translate-y-10'"
      >
        <div v-for="r in 5" :key="r" class="flex flex-col gap-1">
          <div v-for="c in 5" :key="c" 
            class="w-8 h-8 rounded text-[8px] flex items-center justify-center"
            :class="(r+c) % 3 === 0 ? 'bg-red-500 text-white' : (r+c) % 2 === 0 ? 'bg-red-200 text-red-900' : 'bg-red-50 text-red-300'"
          >
            ΔP
          </div>
        </div>
      </div>
    </div>

    <div class="mt-8 text-center h-12 flex flex-col items-center justify-center italic text-gray-500 text-sm max-w-md">
        <div v-if="clicks === 0">Base Sequence: Input for the model.</div>
        <div v-if="clicks === 1">1. Perturbation: Mutate one position (e.g., Y → A).</div>
        <div v-if="clicks === 2">2. Sensitivity: How does this change propagate?</div>
        <div v-if="clicks === 3">3. Jacobian: The matrix of all marginal effects $\frac{\partial P_j}{\partial x_i}$.</div>
    </div>
  </div>
</template>
