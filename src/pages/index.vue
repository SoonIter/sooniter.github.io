<script setup lang="ts">

</script>

<template>
  <main id="main" w-full md="w-full flex-row justify-center" flex flex-col gap-6>
    <aside md="h-full w-80 bg-opacity-0">
      <ProfileCard />
    </aside>
    <div md="flex-1" md:max-w-130 flex flex-col gap-4>
      <Suspense>
        <Tabs />
      </Suspense>
      <Footer :center="false" />
    </div>
  </main>
</template>

<style>

</style>

<route lang="yaml">
meta:
  layout: home
</route>
