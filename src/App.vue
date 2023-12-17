<template>
  <div class="flex flex-col min-h-screen font Roboto bg-weather-primary">
    <!-- Это компонеты которые были импортные на главную app страницу -->
    <SiteNavigation />
    <RouterView />
  </div>
</template>

<script setup>
import { RouterView } from "vue-router";
import SiteNavigation from "./components/SiteNavigation.vue";
</script>

<style lang="scss" scoped></style>
