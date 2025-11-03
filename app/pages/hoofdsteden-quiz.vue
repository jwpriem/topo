<script setup lang="ts">
import Quiz from '~/components/Quiz.vue'

type Province = { name: string; capital: string }

const provinces: Province[] = [
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

function shuffle<T>(a: T[]): T[] {
  const arr = a.slice()
  for (let i = arr.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[arr[i], arr[j]] = [arr[j], arr[i]]
  }
  return arr
}

const allProvinces = provinces.map(p => p.name)

const questions = provinces.map(p => {
  const distractors = shuffle(allProvinces.filter(n => n !== p.name)).slice(0, 3)
  const options = shuffle([
    { label: p.name, isCorrect: true },
    ...distractors.map(d => ({ label: d, isCorrect: false })),
  ])
  return {
    prompt: `In welke provincie ligt ${p.capital}?`,
    options,
  }
})

const seed = useState('quiz-seed', () => Math.random())
</script>

<template>
  <div class="min-h-screen bg-gradient-to-b from-slate-50 to-white flex items-center justify-center p-4">
    <Quiz title="Hoofdsteden → Provincies 🧭"
          :questions="questions"
          :seed="seed"
          :timer-enabled="true"
          :timer-seconds="10" />
  </div>
</template>
