<script setup lang="ts">
import { ref, computed, onBeforeUnmount } from 'vue'

type Option = { label: string; isCorrect: boolean }
type Question = { prompt: string; options: Option[] }

const props = defineProps<{
  title: string
  questions: Question[]
  shuffleOptions?: boolean
  timerEnabled?: boolean
  timerSeconds?: number
  autoAdvanceDelayMs?: number
  /** Optionele seed om volgorde te determineren (voorkomt SSR hydration mismatch) */
  seed?: number
}>()

const emit = defineEmits<{
  (e: 'complete', payload: { score: number; total: number }): void
}>()

/* ---------------- Timer-instellingen ---------------- */
function coerceTimerEnabled(val: unknown): boolean {
  if (val === false) return false
  if (typeof val === 'string') return val.toLowerCase().trim() !== 'false'
  return true
}
const timerEnabled = computed<boolean>(() => coerceTimerEnabled((props as any).timerEnabled))
const timerTotalMs = computed<number>(() => {
  const sec = Number(props.timerSeconds ?? 10)
  return Math.max(0, Math.floor(sec * 1000))
})
const autoAdvanceDelayMs = computed(() => props.autoAdvanceDelayMs ?? 800)

/* ------------- Deterministische shuffle -------------- */
function mulberry32(s: number) {
  return function () {
    let t = (s += 0x6D2B79F5)
    t = Math.imul(t ^ (t >>> 15), t | 1)
    t ^= t + Math.imul(t ^ (t >>> 7), t | 61)
    return ((t ^ (t >>> 14)) >>> 0) / 4294967296
  }
}
function shuffleSeed<T>(arr: T[], rnd: () => number): T[] {
  const a = arr.slice()
  for (let i = a.length - 1; i > 0; i--) {
    const j = Math.floor(rnd() * (i + 1))
    ;[a[i], a[j]] = [a[j], a[i]]
  }
  return a
}
function hashString(s: string) {
  let h = 5381
  for (let i = 0; i < s.length; i++) h = ((h << 5) + h) ^ s.charCodeAt(i)
  return Math.abs(h)
}
function deriveSeedFromQuestions(qs: Question[]) {
  const key = qs
      .map(q => q.prompt + '|' + q.options.map(o => `${o.label}:${o.isCorrect ? 1 : 0}`).join(','))
      .join('||')
  return hashString(key) || 1
}

/* --------------------- State ------------------------ */
const started = ref(false)
const prepared = ref<Question[]>([])
const current = ref(0)
const selectedIndex = ref<number | null>(null)
const answered = ref(false)
const isCorrect = ref<boolean | null>(null)
const score = ref(0)
const finished = ref(false)

const remainingMs = ref(0)
const timerId = ref<number | null>(null)
const timeoutId = ref<number | null>(null)
const timedOut = ref(false)

/** Resultaten voor samenvatting */
const totalTimeSec = ref(0) // totale tijd over alle vragen (incl. timeouts)
const wrongs = ref<{ q: string; given: string; correct: string; timeSec: number }[]>([])

const progressText = computed(() => `${Math.min(current.value + 1, prepared.value.length)} / ${prepared.value.length}`)
const timePercent = computed(() => {
  if (!timerEnabled.value || timerTotalMs.value <= 0) return 0
  return Math.max(0, Math.min(100, (remainingMs.value / timerTotalMs.value) * 100))
})
const secondsLeft = computed(() => Math.ceil(remainingMs.value / 1000))
const avgTimeSec = computed(() =>
    prepared.value.length ? +(totalTimeSec.value / prepared.value.length).toFixed(1) : 0
)

/* ------------------- Timer helpers ------------------ */
function clearTimers() {
  if (timerId.value) {
    clearInterval(timerId.value)
    timerId.value = null
  }
  if (timeoutId.value) {
    clearTimeout(timeoutId.value)
    timeoutId.value = null
  }
}
function startTimer() {
  clearTimers()
  timedOut.value = false
  if (!timerEnabled.value || timerTotalMs.value <= 0) return
  remainingMs.value = timerTotalMs.value
  timerId.value = window.setInterval(() => {
    remainingMs.value = Math.max(0, remainingMs.value - 100)
    if (remainingMs.value <= 0) {
      clearInterval(timerId.value!)
      timerId.value = null
      onTimeExpired()
    }
  }, 100)
}

/** Hulp om juiste antwoordlabel te vinden voor huidige vraag */
function getCorrectLabel(): string {
  const opt = prepared.value[current.value].options.find(o => o.isCorrect)
  return opt ? opt.label : ''
}

/** Log resultaat + tijd voor huidige vraag */
function recordResult(isRight: boolean, givenLabel: string, timedOutFlag: boolean) {
  // tijd die deze vraag gekost heeft (sec). Bij timeout = volle tijd.
  const elapsedMs = timedOutFlag ? timerTotalMs.value : (timerTotalMs.value - remainingMs.value)
  const timeSec = +(elapsedMs / 1000).toFixed(1)
  totalTimeSec.value += timeSec

  if (!isRight) {
    wrongs.value.push({
      q: prepared.value[current.value].prompt,
      given: givenLabel,
      correct: getCorrectLabel(),
      timeSec
    })
  }
}

function onTimeExpired() {
  if (answered.value || finished.value) return
  timedOut.value = true
  answered.value = true
  isCorrect.value = false
  // niets gekozen -> markeer als fout met "tijd verstreken"
  recordResult(false, '— tijd verstreken', true)
  timeoutId.value = window.setTimeout(() => {
    nextQuestion()
  }, autoAdvanceDelayMs.value)
}

/* -------------------- Prepare ----------------------- */
function resetState() {
  current.value = 0
  selectedIndex.value = null
  answered.value = false
  isCorrect.value = null
  score.value = 0
  finished.value = false
  timedOut.value = false
  clearTimers()

  // samenvatting resetten
  totalTimeSec.value = 0
  wrongs.value = []
}
function prepareOnce() {
  const baseSeed = (typeof props.seed === 'number' ? props.seed : deriveSeedFromQuestions(props.questions))
  const rnd = mulberry32(baseSeed)

  prepared.value = (props.questions || []).map(q => ({
    prompt: q.prompt,
    options: props.shuffleOptions === false ? q.options : shuffleSeed(q.options, rnd),
  }))

  resetState()
}
prepareOnce()

/* ------------------- Interactie --------------------- */
function startQuiz() {
  started.value = true
  startTimer()
}
function selectOption(idx: number) {
  if (answered.value || finished.value) return
  selectedIndex.value = idx
  answered.value = true

  const chosen = prepared.value[current.value].options[idx]
  isCorrect.value = chosen.isCorrect
  if (chosen.isCorrect) {
    score.value++
    recordResult(true, chosen.label, false)
  } else {
    recordResult(false, chosen.label, false)
  }

  clearTimers()
  timeoutId.value = window.setTimeout(() => {
    nextQuestion()
  }, autoAdvanceDelayMs.value)
}
function nextQuestion() {
  if (finished.value) return
  if (current.value < prepared.value.length - 1) {
    current.value++
    selectedIndex.value = null
    answered.value = false
    isCorrect.value = null
    timedOut.value = false
    startTimer()
  } else {
    finished.value = true
    clearTimers()
    emit('complete', { score: score.value, total: prepared.value.length })
  }
}
function restart() {
  started.value = false
  prepareOnce()
}

onBeforeUnmount(() => clearTimers())
</script>

<template>
  <div class="w-full max-w-2xl">
    <div class="bg-white rounded-2xl shadow-lg p-6 md:p-8 border border-slate-100 text-center">
      <!-- Startscherm -->
      <div v-if="!started && !finished" class="py-12">
        <h1 class="text-3xl font-bold mb-4 text-slate-800">{{ title }}</h1>
        <p class="text-slate-600 mb-6">
          Je krijgt {{ (timerTotalMs/1000) }} seconden per vraag. Beantwoord zo snel mogelijk!
        </p>
        <button
            @click="startQuiz"
            class="px-6 py-3 rounded-xl bg-emerald-600 text-white font-medium text-lg hover:bg-emerald-700 transition"
        >
          Start quiz
        </button>
      </div>

      <!-- Einde / Samenvatting -->
      <div v-else-if="finished" class="py-8 text-left">
        <p class="text-2xl font-semibold text-slate-800 text-center">Klaar! 🎉</p>

        <div class="mt-4 grid gap-2 sm:grid-cols-3 text-sm">
          <div class="bg-slate-50 rounded-lg p-4 border border-slate-100">
            <div class="text-slate-500">Score</div>
            <div class="text-xl font-semibold">{{ score }} / {{ prepared.length }}</div>
          </div>
          <div class="bg-slate-50 rounded-lg p-4 border border-slate-100">
            <div class="text-slate-500">Totaaltijd</div>
            <div class="text-xl font-semibold">{{ totalTimeSec.toFixed(1) }}s</div>
          </div>
          <div class="bg-slate-50 rounded-lg p-4 border border-slate-100">
            <div class="text-slate-500">Gemiddeld / vraag</div>
            <div class="text-xl font-semibold">{{ avgTimeSec }}s</div>
          </div>
        </div>

        <div class="mt-8">
          <h3 class="text-lg font-semibold text-slate-800 mb-3">Fout beantwoorde vragen</h3>
          <div v-if="wrongs.length === 0" class="text-slate-500">
            Mooi! Je had geen fouten. ✅
          </div>

          <div v-else class="overflow-x-auto">
            <table class="min-w-full text-sm">
              <thead>
              <tr class="text-left text-slate-500">
                <th class="py-2 pr-4">Vraag</th>
                <th class="py-2 pr-4">Jouw antwoord</th>
                <th class="py-2 pr-4">Juiste antwoord</th>
                <th class="py-2 pr-4 text-right">Tijd (s)</th>
              </tr>
              </thead>
              <tbody>
              <tr v-for="(w, i) in wrongs" :key="i" class="border-t border-slate-100">
                <td class="py-2 pr-4 align-top">{{ w.q }}</td>
                <td class="py-2 pr-4 align-top text-rose-600 font-medium">
                  {{ w.given }}
                </td>
                <td class="py-2 pr-4 align-top text-emerald-700 font-medium">
                  {{ w.correct }}
                </td>
                <td class="py-2 pr-4 align-top text-right tabular-nums">
                  {{ w.timeSec.toFixed(1) }}
                </td>
              </tr>
              </tbody>
            </table>
          </div>
        </div>

        <div class="mt-8 text-center">
          <button
              @click="restart"
              class="px-4 py-2 rounded-xl bg-emerald-600 text-white text-sm font-medium hover:bg-emerald-700 transition"
          >
            Opnieuw spelen
          </button>
        </div>
      </div>

      <!-- Quiz actief -->
      <template v-else>
        <!-- Header -->
        <div class="flex items-center justify-between mb-6">
          <h2 class="text-lg md:text-xl font-semibold">{{ progressText }}</h2>
          <div v-if="timerEnabled" class="flex items-center gap-2 text-sm font-medium tabular-nums">
            ⏱️ <span>{{ secondsLeft }}s</span>
          </div>
        </div>

        <!-- Timer balk -->
        <div v-if="timerEnabled" class="w-full h-2 bg-slate-100 rounded-full overflow-hidden mb-6">
          <div
              class="h-full transition-[width,background-color] duration-100"
              :class="[
              timePercent > 50 ? 'bg-sky-500' :
              timePercent > 20 ? 'bg-amber-500' : 'bg-rose-500'
            ]"
              :style="{ width: timePercent + '%' }"
          />
        </div>

        <!-- Vraag -->
        <p class="text-lg md:text-xl font-medium mb-6">
          {{ prepared[current].prompt }}
        </p>

        <!-- Antwoorden -->
        <div class="grid gap-3 text-left">
          <button
              v-for="(opt, idx) in prepared[current].options"
              :key="opt.label + idx"
              @click="selectOption(idx)"
              class="group relative w-full rounded-xl border p-4 md:p-5 transition
                   focus:outline-none focus:ring-2 focus:ring-offset-2
                   disabled:opacity-60 disabled:cursor-not-allowed"
              :class="[
              'border-slate-200 bg-white hover:border-slate-300',
              selectedIndex === idx && answered && opt.isCorrect ? 'border-emerald-500 ring-emerald-200' : '',
              selectedIndex === idx && answered && !opt.isCorrect ? 'border-rose-500 ring-rose-200' : '',
              selectedIndex === idx && !answered ? 'border-sky-300 ring-1 ring-sky-200' : '',
              timedOut && opt.isCorrect ? 'ring-1 ring-emerald-200' : ''
            ]"
              :disabled="answered"
          >
            <div class="flex items-center justify-between">
              <span class="text-base md:text-lg">{{ opt.label }}</span>
              <span v-if="answered && selectedIndex === idx && opt.isCorrect" class="ml-3">✅</span>
              <span v-else-if="answered && selectedIndex === idx && !opt.isCorrect" class="ml-3">❌</span>
              <span v-else-if="answered && timedOut && opt.isCorrect" class="ml-3 text-emerald-600">✔️</span>
            </div>
          </button>
        </div>

        <!-- Score -->
        <div class="mt-6 text-sm text-slate-600 text-right">
          Score: <span class="font-semibold text-slate-900">{{ score }}</span>
        </div>
      </template>
    </div>
  </div>
</template>