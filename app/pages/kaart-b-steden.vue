<script setup lang="ts">
import Quiz from '~/components/Quiz.vue'

type ProvinceCities = {
  province: string
  cities: string[]
}

const provinces: ProvinceCities[] = [
  {
    province: 'Friesland',
    cities: ['Leeuwarden', 'Sneek', 'Drachten', 'Heerenveen'],
  },
  {
    province: 'Groningen',
    cities: ['Groningen', 'Delfzijl', 'Veendam'],
  },
  {
    province: 'Drenthe',
    cities: ['Assen', 'Meppel', 'Hoogeveen', 'Emmen'],
  },
]

// 🔧 setting
const difficult = ref(false)
const pendingDifficult = ref<boolean | null>(null)
const showConfirm = ref(false)

const quizKey = computed(() => `quiz-${difficult.value}`)

// localStorage
onMounted(() => {
  const stored = localStorage.getItem('quiz-difficult')
  if (stored !== null) {
    difficult.value = stored === 'true'
  }
})

watch(difficult, value => {
  localStorage.setItem('quiz-difficult', String(value))
})

function requestDifficultyChange(value: boolean) {
  if (value === difficult.value) return

  pendingDifficult.value = value
  showConfirm.value = true
}

function confirmChange() {
  if (pendingDifficult.value !== null) {
    difficult.value = pendingDifficult.value
  }
  pendingDifficult.value = null
  showConfirm.value = false
}

function cancelChange() {
  pendingDifficult.value = null
  showConfirm.value = false
}


function shuffle<T>(a: T[]): T[] {
  const arr = a.slice()
  for (let i = arr.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
      ;[arr[i], arr[j]] = [arr[j], arr[i]]
  }
  return arr
}

function formatCities(cities: string[]) {
  return cities.join(', ')
}

function uniqueByLabel(
  options: { label: string; isCorrect: boolean }[]
) {
  const seen = new Set<string>()
  return options.filter(o => {
    if (seen.has(o.label)) return false
    seen.add(o.label)
    return true
  })
}

// 🔥 moeilijke antwoordopties
function createDifficultOptions(
  correctCities: string[],
  allCities: string[],
  optionCount = 6
) {
  const cityCount = correctCities.length
  const sortedCorrect = correctCities.slice().sort().join(', ')

  const options: { label: string; isCorrect: boolean }[] = [
    {
      label: sortedCorrect,
      isCorrect: true,
    },
  ]

  while (options.length < optionCount) {
    const mixed = shuffle(allCities)
      .slice(0, cityCount)
      .sort()
      .join(', ')

    if (mixed === sortedCorrect) continue

    options.push({
      label: mixed,
      isCorrect: false,
    })
  }

  return uniqueByLabel(options)
}

const allCities = provinces.flatMap(p => p.cities)

const questions = computed(() =>
  provinces.map(p => {
    let options

    if (difficult.value) {
      options = createDifficultOptions(
        p.cities,
        allCities,
        6
      )
    } else {
      const correctOption = {
        label: formatCities(p.cities),
        isCorrect: true,
      }

      const distractors = shuffle(
        provinces.filter(o => o.province !== p.province)
      )
        .slice(0, 2)
        .map(o => ({
          label: formatCities(o.cities),
          isCorrect: false,
        }))

      options = [correctOption, ...distractors]
    }

    return {
      prompt: `Welke steden horen bij ${p.province}?`,
      options: shuffle(options),
    }
  })
)

const seed = useState('quiz-seed', () => Math.random())
</script>

<template>
  <div
    class="min-h-screen bg-gradient-to-b from-slate-50 to-white flex flex-col items-center justify-center p-4 space-y-6">
    <ToggleSwitch :model-value="difficult" label="Niveau moeilijk?"
      @toggle-request="requestDifficultyChange(!difficult)" />
    <Quiz title="Provincies → Steden 🏙️" :key="quizKey" :questions="questions" :seed="seed" :timer-enabled="true"
      :timer-seconds="12" />
  </div>

  <!-- confirm modal -->
  <div v-if="showConfirm" class="fixed inset-0 z-50 flex items-center justify-center bg-black/40">
    <div class="bg-white rounded-2xl shadow-xl w-80 p-6 text-center">
      <h2 class="text-lg font-semibold mb-2">
        Weet je het zeker?
      </h2>

      <p class="text-sm text-gray-600 mb-6">
        De quiz wordt opnieuw gestart.
      </p>

      <div class="flex gap-3">


        <button
          class="px-4 py-2 rounded-xl bg-emerald-600 text-white text-sm font-medium hover:bg-emerald-700 transition"
          @click="confirmChange">
          Ja
        </button>

        <button class="px-4 py-2 rounded-xl bg-rose-600 text-white text-sm font-medium hover:bg-rose-700 transition"
          @click="cancelChange">
          Nee
        </button>
      </div>
    </div>
  </div>

</template>
