<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps<{
  clicks: number
}>()

const aaColors: Record<string, string> = {
  // Hydrophobic - Vibrant Greens/Teals
  'M': 'bg-emerald-500 text-white', 'V': 'bg-emerald-600 text-white', 
  'I': 'bg-teal-500 text-white', 'L': 'bg-teal-600 text-white', 
  'A': 'bg-green-400 text-black', 'F': 'bg-emerald-700 text-white',
  // Positively Charged - Bold Blues
  'K': 'bg-blue-600 text-white', 'R': 'bg-indigo-600 text-white',
  // Negatively Charged - Bold Reds
  'D': 'bg-red-600 text-white', 'E': 'bg-rose-600 text-white',
  // Polar Uncharged - Oranges/Yellows
  'S': 'bg-amber-400 text-black', 'T': 'bg-orange-500 text-white', 
  'N': 'bg-yellow-400 text-black', 'Q': 'bg-orange-400 text-black',
  'G': 'bg-slate-300 text-black',
  '-': 'bg-transparent text-gray-300'
}

const rawSequences = [
  "MKVILLLVPAMLGITVMAFSGSTR",
  "ILLLVPAMLGITVMAFS",
  "GAMKVILLLVPAMLGITVMAFSGSTR",
  "VLLVPAMLGITVMAFSGSTR",
  "MKVILLLVPAMLGITVMAFSGSTR"
]

const alignedSequences = [
  "--MKVILLLVPAMLGITVMAFSGSTR",
  "--MK-ILLLVPAMLGITVMAFS----",
  "GAMKVILLLVPAMLGIT---FSGSTR",
  "---MKV-LL-VPAMLGITVMAFSGSTR",
  "--MKVILLLVPAMLGITVMAFSGSTR"
]

const isAligned = computed(() => props.clicks >= 1)
const currentSeqs = computed(() => isAligned.value ? alignedSequences : rawSequences)

const getCellClass = (char: string) => {
  if (!isAligned.value) return 'bg-gray-100 text-gray-800'
  return aaColors[char] || 'bg-gray-50'
}
</script>

<template>
  <div class="flex flex-col items-center justify-center w-full bg-white rounded-xl p-8 shadow-md border border-gray-200 font-mono">
    <div class="space-y-2">
      <div v-for="(seq, idx) in currentSeqs" :key="idx" class="flex items-center space-x-2">
        <div class="w-12 text-xs font-bold text-gray-400">Seq {{ idx + 1 }}</div>
        <div class="flex">
          <div 
            v-for="(char, cIdx) in seq.split('')" 
            :key="cIdx"
            class="w-6 h-6 flex items-center justify-center text-[9px] font-bold border border-white transition-all duration-700"
            :class="getCellClass(char)"
          >
            {{ char }}
          </div>
        </div>
      </div>
    </div>

    <div class="mt-8 text-center h-8 flex items-center justify-center italic text-gray-500 text-sm">
        <div v-if="clicks === 0">Unaligned sequences: Hidden patterns.</div>
        <div v-if="clicks >= 1">Aligned sequences (MSA): Evolutionary relationships exposed.</div>
    </div>
  </div>
</template>
