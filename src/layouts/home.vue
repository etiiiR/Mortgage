<template>
  <main class="px-4 py-10 text-center text-gray-700 dark:text-gray-200">
    <Suspense>
      <!-- component with nested async dependencies -->
      <RouterView />

      <!-- loading state via #fallback slot -->
      <template #fallback>
        Loading...
      </template>
    </Suspense>
    <Footer />
    <div class="mt-5 mx-auto text-center opacity-75 dark:opacity-50 text-sm">
      [Home Layout]
    </div>
  </main>
</template>

