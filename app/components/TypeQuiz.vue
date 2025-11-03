<script setup lang="ts">
import { ref, computed } from 'vue'

type Question = {
  prompt: string
  answer: string
}

// Props
const props = defineProps<{
  title: string
  questions: Question[]
}>()

// State
const current = ref(0)
const userAnswer = ref('')
const feedback = ref<'correct' | 'incorrect' | null>(null)
const score = ref(0)
const finished = ref(false)

// Helpers
function normalize(str: string) {
  return str
      .toLowerCase()
      .normalize('NFD')
      .replace(/[\u0300-\u036f]/g, '') // verwijder accenten
      .replace(/[^a-z0-9]/g, '') // verwijder spaties en leestekens
}

// Bereken Levenshtein-afstand voor fouttolerantie
function levenshtein(a: string, b: string) {
  const m = a.length
  const n = b.length
  const dp = Array.from({ length: m + 1 }, () => Array(n + 1).fill(0))
  for (let i = 0; i <= m; i++) dp[i][0] = i
  for (let j = 0; j <= n; j++) dp[0][j] = j
  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      const cost = a[i - 1] === b[j - 1] ? 0 : 1
      dp[i][j] = Math.min(dp[i - 1][j] + 1, dp[i][j - 1] + 1, dp[i - 1][j - 1] + cost)
    }
  }
  return dp[m][n]
}

// Controleer antwoord
function checkAnswer() {
  const correct = props.questions[current.value].answer
  const normalizedUser = normalize(userAnswer.value)
  const normalizedCorrect = normalize(correct)
  const distance = levenshtein(normalizedUser, normalizedCorrect)

  const isCloseEnough = distance <= 1
  if (isCloseEnough) {
    feedback.value = 'correct'
    score.value++
  } else {
    feedback.value = 'incorrect'
  }
}

function nextQuestion() {
  if (current.value < props.questions.length - 1) {
    current.value++
    feedback.value = null
    userAnswer.value = ''
  } else {
    finished.value = true
  }
}

const progressText = computed(() => `${current.value + 1} / ${props.questions.length}`)
</script>

<template>
  <div class="w-full max-w-2xl">
    <div class="bg-white rounded-2xl shadow-lg p-6 md:p-8 border border-slate-100">
      <div class="flex items-center justify-between mb-6">
        <h1 class="text-xl md:text-2xl font-semibold tracking-tight">{{ title }}</h1>
        <div v-if="!finished" class="text-sm text-slate-600">
          Vraag <span class="font-medium text-slate-900">{{ progressText }}</span>
        </div>
      </div>

      <div v-if="finished" class="text-center py-12">
        <p class="text-2xl font-semibold text-slate-800">Klaar! 🎉</p>
        <p class="mt-2 text-slate-600">Je score: <span class="font-semibold">{{ score }}</span> / {{ questions.length }}</p>
        <button
            @click="() => location.reload()"
            class="mt-6 px-4 py-2 rounded-xl bg-emerald-600 text-white text-sm font-medium hover:bg-emerald-700 transition"
        >
          Opnieuw spelen
        </button>
      </div>

      <template v-else>
        <p class="text-lg md:text-xl font-medium mb-6">
          {{ questions[current].prompt }}
        </p>

        <input
            v-model="userAnswer"
            type="text"
            class="w-full rounded-lg border border-slate-300 px-4 py-2 text-base focus:ring-2 focus:ring-sky-300 outline-none"
            placeholder="Typ je antwoord hier..."
            :disabled="feedback !== null"
            @keyup.enter="feedback ? nextQuestion() : checkAnswer()"
        />

        <!-- Feedback -->
        <div v-if="feedback" class="mt-4">
          <p
              v-if="feedback === 'correct'"
              class="text-emerald-600 font-medium flex items-center gap-2"
          >
            ✅ Goed gedaan!
          </p>
          <p
              v-else
              class="text-rose-600 font-medium flex items-center gap-2"
          >
            ❌ Fout! Het juiste antwoord was:
            <span class="font-semibold">{{ questions[current].answer }}</span>
          </p>
        </div>

        <div class="mt-6 flex justify-between items-center">
          <div class="text-sm text-slate-600">
            Score: <span class="font-semibold text-slate-900">{{ score }}</span>
          </div>

          <button
              v-if="!feedback"
              @click="checkAnswer"
              :disabled="!userAnswer.trim()"
              class="px-4 py-2 rounded-xl bg-slate-900 text-white text-sm font-medium disabled:opacity-40 hover:bg-slate-800 transition"
          >
            Nakijken
          </button>

          <button
              v-else
              @click="nextQuestion"
              class="px-4 py-2 rounded-xl bg-emerald-600 text-white text-sm font-medium hover:bg-emerald-700 transition"
          >
            Volgende
          </button>
        </div>
      </template>
    </div>
  </div>
</template>
