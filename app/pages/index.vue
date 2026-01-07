<template>
  <section class="min-h-[70vh] flex flex-col items-center justify-center px-6 py-6">
    <div class="text-center mb-10">
      <h2 class="text-4xl font-bold text-slate-700">Welkom</h2>
      <p class="mt-3 text-slate-500">Kies een quiz om te starten.</p>
    </div>

    <!-- Tegels -->
    <div class="grid gap-6 sm:grid-cols-2 max-w-3xl w-full">
      <!-- Provincie Quiz -->
      <NuxtLink to="/provincie-quiz"
        class="group bg-white rounded-2xl shadow hover:shadow-lg border border-slate-100 p-6 transition transform hover:-translate-y-1 flex flex-col items-center text-center">
        <div
          class="w-16 h-16 mb-4 flex items-center justify-center rounded-full bg-sky-100 text-sky-600 text-3xl font-bold">
          🗺️
        </div>
        <h3 class="text-xl font-semibold text-slate-800 group-hover:text-sky-600">Provincie Quiz</h3>
        <p class="mt-2 text-slate-500 text-sm">Weet jij van elke provincie wat de hoofdstad is?</p>
      </NuxtLink>

      <!-- Hoofdsteden Quiz -->
      <NuxtLink to="/hoofdsteden-quiz"
        class="group bg-white rounded-2xl shadow hover:shadow-lg border border-slate-100 p-6 transition transform hover:-translate-y-1 flex flex-col items-center text-center">
        <div
          class="w-16 h-16 mb-4 flex items-center justify-center rounded-full bg-emerald-100 text-emerald-600 text-3xl font-bold">
          🧭
        </div>
        <h3 class="text-xl font-semibold text-slate-800 group-hover:text-emerald-600">Hoofdsteden Quiz</h3>
        <p class="mt-2 text-slate-500 text-sm">Weet jij in welke provincie een hoofdstad ligt?</p>
      </NuxtLink>

      <NuxtLink to="/algemene-typequiz"
        class="group bg-white rounded-2xl shadow hover:shadow-lg border border-slate-100 p-6 transition transform hover:-translate-y-1 flex flex-col items-center text-center">
        <div
          class="w-16 h-16 mb-4 flex items-center justify-center rounded-full bg-indigo-100 text-indigo-600 text-3xl font-bold">
          🇳🇱
        </div>
        <h3 class="text-xl font-semibold text-slate-800 group-hover:text-indigo-600">Algemene Typquiz</h3>
        <p class="mt-2 text-slate-500 text-sm">Provincies en hoofdsteden door elkaar.</p>
      </NuxtLink>

      <NuxtLink to="/steden-noord"
        class="group bg-white rounded-2xl shadow hover:shadow-lg border border-slate-100 p-6 transition transform hover:-translate-y-1 flex flex-col items-center text-center">
        <div
          class="w-16 h-16 mb-4 flex items-center justify-center rounded-full bg-rose-100 text-rose-600 text-3xl font-bold">
          🏙️
        </div>
        <h3 class="text-xl font-semibold text-slate-800 group-hover:text-rose-600">Steden (Noord)</h3>
        <p class="mt-2 text-slate-500 text-sm">Kies de juiste steden bij de provincie.</p>
      </NuxtLink>

    </div>
  </section>
</template>

<script setup lang="ts">
// geen logica nodig
</script>
