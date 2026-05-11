<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps<{
  clicks: number
}>()

// Animation states
const isJoined = computed(() => props.clicks >= 2)
const isHighlighted = computed(() => props.clicks >= 1)

/**
 * COORDINATE CALCULATION (Expansion)
 * Center of SVG is x=400.
 * 
 * Separated Phase:
 * We want a wide gap of 300px.
 * AA1 C at x=250 => translate x = 75 (250 - 175)
 * AA2 N at x=550 => translate x = 480 (550 - 70)
 * Midpoint is 400.
 * 
 * Joined Phase:
 * We want a clear gap of 120px for the bond.
 * AA1 C at x=340 => translate x = 165 (340 - 175)
 * AA2 N at x=460 => translate x = 390 (460 - 70)
 */
const aa1X = computed(() => isJoined.value ? 165 : 75)
const aa2X = computed(() => isJoined.value ? 390 : 480)

// The green bond bridges the 120px gap (from x=340 to x=460)
// We add padding so it doesn't touch the letters C and N
const bondStartX = 355
const bondEndX = 445
</script>

<template>
  <div class="flex flex-col items-center justify-center w-full bg-white rounded-xl p-4 shadow-inner border border-gray-100 overflow-hidden min-h-[500px]">
    
    <svg width="800" height="500" viewBox="0 0 800 500" class="font-serif">
      
      <!-- --- AMINO ACID 1 (Left Part) --- -->
      <g 
        class="transition-transform duration-1000 ease-in-out"
        :style="{ transform: `translate(${aa1X}px, 100px)` }"
      >
        <!-- Static Parts -->
        <text x="0" y="100" class="label">H₂N</text>
        <line x1="45" y1="95" x2="80" y2="95" class="chem-line" />
        
        <!-- C-alpha with rock-solid subscript -->
        <text x="85" y="100" class="label">C</text>
        <text x="98" y="108" class="alpha-label">α</text>
        
        <line x1="97" y1="75" x2="97" y2="45" class="chem-line" />
        <text x="90" y="40" class="label">R₁</text>
        <line x1="97" y1="105" x2="97" y2="135" class="chem-line" />
        <text x="92" y="155" class="label">H</text>

        <line x1="115" y1="95" x2="150" y2="95" class="chem-line" />

        <!-- Carboxyl C and O (Always visible) -->
        <text x="175" y="95" class="label" :class="{ 'blue-text': isHighlighted && !isJoined }">C</text>
        <line x1="183" y1="105" x2="183" y2="135" class="chem-line" :class="{ 'blue-line': isHighlighted && !isJoined }" />
        <line x1="188" y1="105" x2="188" y2="135" class="chem-line" :class="{ 'blue-line': isHighlighted && !isJoined }" />
        <text x="180" y="155" class="label" :class="{ 'blue-text': isHighlighted && !isJoined }">O</text>

        <!-- Reactive OH (Leaves) -->
        <g class="transition-all duration-500" :style="{ opacity: isJoined ? 0 : 1, transform: isJoined ? 'translate(0, -30px)' : 'translate(0,0)' }">
          <line x1="195" y1="95" x2="225" y2="95" class="chem-line" :class="{ 'blue-line': isHighlighted }" />
          <text x="230" y="95" class="label" :class="{ 'blue-text': isHighlighted }">OH</text>
          
          <rect v-if="isHighlighted && !isJoined" x="155" y="40" width="120" height="130" class="box-blue" />
          <text v-if="isHighlighted && !isJoined" x="175" y="195" class="label blue-text text-sm font-bold">Carboxyl</text>
        </g>
      </g>

      <!-- --- PLUS SIGN (Centered in the 300px initial gap) --- -->
      <text 
        x="400" y="195" font-size="40" fill="#cbd5e1" text-anchor="middle"
        class="transition-opacity duration-500"
        :style="{ opacity: isJoined ? 0 : 1 }"
      >+</text>

      <!-- --- AMINO ACID 2 (Right Part) --- -->
      <g 
        class="transition-all duration-1000 ease-in-out"
        :style="{ transform: `translate(${aa2X}px, 100px)` }"
      >
        <!-- Highlight Box (Amino) -->
        <rect v-if="isHighlighted && !isJoined" x="0" y="40" width="110" height="130" class="box-red" />
        <text v-if="isHighlighted && !isJoined" x="25" y="195" class="label red-text text-sm font-bold">Amino</text>

        <!-- Reactive H that leaves -->
        <g class="transition-all duration-500" :style="{ opacity: isJoined ? 0 : 1, transform: isJoined ? 'translate(0, -30px)' : 'translate(0,0)' }">
          <text x="15" y="95" class="label" :class="{ 'red-text': isHighlighted }">H</text>
          <line x1="35" y1="95" x2="65" y2="95" class="chem-line" :class="{ 'red-line': isHighlighted }" />
        </g>

        <!-- N and vertical H (Stay) -->
        <text x="70" y="95" class="label" :class="{ 'red-text': isHighlighted && !isJoined }">N</text>
        <line x1="78" y1="105" x2="78" y2="135" class="chem-line" :class="{ 'red-line': isHighlighted && !isJoined }" />
        <text x="73" y="155" class="label" :class="{ 'red-text': isHighlighted && !isJoined }">H</text>

        <!-- AA2 Tail -->
        <line x1="90" y1="95" x2="125" y2="95" class="chem-line" />
        <text x="130" y="100" class="label">C</text>
        <text x="143" y="108" class="alpha-label">α</text>
        
        <line x1="142" y1="75" x2="142" y2="45" class="chem-line" />
        <text x="135" y="40" class="label">R₂</text>
        <line x1="142" y1="105" x2="142" y2="135" class="chem-line" />
        <text x="137" y="155" class="label">H</text>

        <line x1="165" y1="95" x2="200" y2="95" class="chem-line" />
        <text x="205" y="100" class="label">COOH</text>
      </g>

      <!-- --- THE NEW PEPTIDE BOND --- -->
      <g 
        class="transition-opacity duration-500 delay-700"
        :style="{ opacity: isJoined ? 1 : 0 }"
      >
        <!-- The bond bridges the gap between C(340) and N(460) -->
        <line :x1="bondStartX" y1="195" :x2="bondEndX" y2="195" class="bold-bond" />
      </g>

      <!-- --- H2O WATER RELEASE --- -->
      <g 
        class="transition-all duration-700 delay-500"
        :style="{ 
          opacity: isJoined ? 1 : 0,
          transform: isJoined ? 'translate(0, 0)' : 'translate(0, -30px)'
        }"
      >
        <g transform="translate(650, 100)">
          <path d="M -40 40 Q 0 0 40 0" fill="none" stroke="#94a3b8" stroke-width="2" stroke-dasharray="4" marker-end="url(#arrow-head)" />
          <text x="50" y="5" class="label" fill="#94a3b8" font-weight="bold">H₂O</text>
        </g>
      </g>

      <defs>
        <marker id="arrow-head" markerWidth="10" markerHeight="7" refX="10" refY="3.5" orient="auto">
          <polygon points="0 0, 10 3.5, 0 7" fill="#94a3b8" />
        </marker>
      </defs>

    </svg>
  </div>
</template>

<style scoped>
.label {
  font-family: 'Times New Roman', serif;
  font-size: 18px;
  fill: black;
}
.alpha-label {
  font-family: 'Times New Roman', serif;
  font-size: 13px;
  fill: black;
}
.blue-text { fill: #0000ff; }
.red-text { fill: #ff0000; }
.box-blue { fill: none; stroke: #0000ff; stroke-width: 1.5; rx: 10; stroke-dasharray: 4; }
.box-red { fill: none; stroke: #ff0000; stroke-width: 1.5; rx: 10; stroke-dasharray: 4; }
.chem-line { stroke: black; stroke-width: 1.5; }
.blue-line { stroke: #0000ff; }
.red-line { stroke: #ff0000; }
.bold-bond { stroke: #10b981; stroke-width: 6; stroke-linecap: round; }
</style>
