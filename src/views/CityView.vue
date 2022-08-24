<template>
  <div>
    <Suspense>
      <AsyncCityView />

      <template #fallback>
        <p
          class="text-white font-Roboto flex items-center justify-center px-2 py-2 w-full"
        >
          Loading...
        </p>
      </template>
    </Suspense>
  </div>
</template>

<script setup>
import AsyncCityView from "../components/AsyncCityView.vue"
</script>
