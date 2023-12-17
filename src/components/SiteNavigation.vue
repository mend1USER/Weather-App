<!--Это навигация нашей страницы которая будет отоброжать
иконку с главным тайтлом которая будет ссылаться на главную страницу.
-->

<template>
  <header class="sticky top-0 bg-weather-primary shadow-lg">
    <nav
      class="container flex flex-col sm:flex-row items-center gap-4 text-white py-6"
    >
      <!-- //навигация и ее стили с иконкой для главной страницы  -->
      <!--роутер линк при помощй него мы определям на какую страницу 
    мы отправим пользовотеля с помощью имени сайт которую мы написали в router -->
      <RouterLink :to="{ name: 'home' }">
        <div class="flex items-center gap-3">
          <i class="fa-solid fa-sun text-2xl"></i>
          <p class="text-2xl">The Local Weather</p>
        </div>
      </RouterLink>

      <!--див для ознокомления приложения с анимацией -->
      <div class="flex gap-3 flex-1 justify-end">
        <i
          class="fa-solid fa-circle-info text-xl hover:text-weather-secondary duration-150 cursor-pointer"
          @click="toggleModal"
        ></i>
        <i
          class="fa-solid fa-plus text-xl hover:text-weather-secondary duration-150 cursor-pointer"
          @click="addCity"
          v-if="route.query.preview"
        ></i>
      </div>
      <!--дочерний компонент в котором есть модальное окно с анимацией-->
      <!--модалактив привязанный к двоиточию что обозначяет что модалактив 
    привязан как атрибут которая будет управлять видимостью модального окна-->

      <!--символ @ которая используется для оброботки события как клоузмодал которую
    я обьявил в компоненте БЭЙЗМОДАЛ с помощью свойсто defineEmits и присвойл ей имя функций 
  ТОГЛМОДАЛ которая при клике будет закрывать окно или открывать ее -->
      <BaseModal :modalActive="modalActive" @close-modal="toggleModal">
        <div class="text-black">
          <h1 class="text-2xl mb-1">About:</h1>
          <p class="mb-4">
            The Local Weather allows you to track the current and future weather
            of cities of your choosing.
          </p>
          <h2 class="text-2xl">How it works:</h2>
          <ol class="list-decimal list-inside mb-4">
            <li>
              Search for your city by entering the name into the search bar.
            </li>
            <li>
              Select a city within the results, this will take you to the
              current weather for your selection.
            </li>
            <li>
              Track the city by clicking on the "+" icon in the top right. This
              will save the city to view at a later time on the home page.
            </li>
          </ol>

          <h2 class="text-2xl">Removing a city</h2>
          <p>
            If you no longer wish to track a city, simply select the city within
            the home page. At the bottom of the page, there will be am option to
            delete the city.
          </p>
        </div>
      </BaseModal>
    </nav>
  </header>
</template>

<script setup>
import { RouterLink, useRoute, useRouter } from "vue-router";

import { uid } from "uid";
import BaseModal from "./BaseModal.vue";
import { ref } from "vue";

const savedCity = ref([]);
const route = useRoute();
const router = useRouter();
//функция для добавления города для начала мы создаем ссылку для хранения городов которую избрали
//с именем savedCity и делаем ее раективной переменной с пустым массивом
//
const addCity = () => {
  //Здесь проверяется наличие записи с ключом "savedCity" в локальном хранилище браузера.
  if (localStorage.getItem("savedCity")) {
    savedCity.value = JSON.parse(localStorage.getItem("savedCity"));
    //Если запись с ключом "savedCity" существует,
    //она извлекается из локального хранилища и преобразуется из JSON в объект.
    // Этот объект присваивается переменной savedCity.value
  }

  const locationObj = {
    id: uid(), // Создаем объект `locationObj` и присваиваем ему свойство "id", в котором содержится уникальный идентификатор, сгенерированный с помощью функции `uid()`
    state: route.params.state, // Свойство "state" в `locationObj` получает значение параметра "state" из текущего маршрута (`route.params.state`).
    city: route.params.city, // Свойство "city" в `locationObj` получает значение параметра "city" из текущего маршрута (`route.params.city`)
    coords: {
      // Создаем объект "coords" внутри `locationObj`, который содержит координаты местоположения.
      lat: route.query.lat, // Свойство "lat" в объекте "coords" получает значение параметра "lat" из текущего запроса (`route.query.lat`).
      lng: route.query.lng, // Свойство "lng" в объекте "coords" получает значение параметра "lng" из текущего запроса (`route.query.lng`).
    },
  };

  savedCity.value.push(locationObj); // Метод `push()` добавляет объект `locationObj` в массив `savedCity.value`. `savedCity.value` вероятно представляет собой переменную состояния Vue.js.
  //Метод `localStorage.setItem()` сохраняет значение массива `savedCity.value` в локальном хранилище браузера. Массив преобразуется в строку с помощью `JSON.stringify()` перед сохранением.
  localStorage.setItem("savedCity", JSON.stringify(savedCity.value));

  // Создаем копию объекта `route.query` и сохраняем ее в переменной `query`.

  let query = Object.assign({}, route.query);
  // Метод `delete` удаляет свойство "preview" из объекта `query`.
  delete query.preview;
  // Добавляем новое свойство "id" в объект `query` и присваиваем ему уникальный идентификатор из `locationObj.id`.
  query.id = locationObj.id;
  // Метод `router.replace()` обновляет текущий маршрут новыми параметрами `query`, включая уникальный идентификатор.

  router.replace({ query });
};
//логика благодаря которой мы можем контролировать
//событие модального окна
//обозночаем модалактив реактивной с помощью реактивной переменной реф обозночяя ее нулл
//и мы будем менять ее с функцией тоглмодал с тру на фолс
const modalActive = ref(null);
const toggleModal = () => {
  modalActive.value = !modalActive.value;
};
</script>
