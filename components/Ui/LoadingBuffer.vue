<template>
  <div
    class="flex flex-col items-center justify-center gap-3 py-10"
    role="status"
    aria-live="polite"
  >
    <img
      src="/jam-animation.gif"
      alt="Loading"
      class="h-20 w-20 rounded-full object-cover shadow-lg animate-pulse"
    />
    <p class="text-gray-500 dark:text-gray-300">Loading...</p>
  </div>
</template>
