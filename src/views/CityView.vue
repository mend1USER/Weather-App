<template>
  <div>
    <!--это специальный компонент внутри вью благодаря которому асинхронные функций можно 
    обробатывать и управляться лучше процессом отоброжения контента для удобной обработки ожидания -->
    <Suspense>
      <!--импортируем компонент с асинхронной функцией и что бы оно прошло успешно обертываем его в тэг <Suspense>-->
      <AsyncCityView />
      <template #fallback>
        <p>Loading</p>
      </template>
    </Suspense>
  </div>
</template>

<script setup>
import AsyncCityView from "../components/AsyncCityView.vue";
</script>
