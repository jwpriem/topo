<script setup lang="ts">
import TypeQuiz from '~/components/TypeQuiz.vue'

const provinces = [
  { name: 'Drenthe',         capital: 'Assen' },
  { name: 'Flevoland',       capital: 'Lelystad' },
  { name: 'Friesland',       capital: 'Leeuwarden' },
  { name: 'Gelderland',      capital: 'Arnhem' },
  { name: 'Groningen',       capital: 'Groningen' },
  { name: 'Limburg',         capital: 'Maastricht' },
  { name: 'Noord-Brabant',   capital: "'s-Hertogenbosch" },
  { name: 'Noord-Holland',   capital: 'Haarlem' },
  { name: 'Overijssel',      capital: 'Zwolle' },
  { name: 'Utrecht',         capital: 'Utrecht' },
  { name: 'Zeeland',         capital: 'Middelburg' },
  { name: 'Zuid-Holland',    capital: 'Den Haag' },
]

// Kleine shuffle helper
function shuffle<T>(arr: T[]): T[] {
  const copy = arr.slice()
  for (let i = copy.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[copy[i], copy[j]] = [copy[j], copy[i]]
  }
  return copy
}

// Bouw gecombineerde vragen
const mixedQuestions = provinces.flatMap(p => [
  { prompt: `Wat is de hoofdstad van ${p.name}?`, answer: p.capital },
  { prompt: `In welke provincie ligt ${p.capital}?`, answer: p.name },
])

// Door elkaar gooien
const questions = shuffle(mixedQuestions)
</script>

<template>
  <div class="min-h-screen flex items-center justify-center bg-gradient-to-b from-slate-50 to-white p-4">
    <TypeQuiz title="Algemene Typquiz 🇳🇱" :questions="questions" />
  </div>
</template>
