<!--компонент для того что бы иттерировать данные избранных городов нажатия на икноку-->
<template>
  <div v-for="city in savedCity" :key="city.id">
    <!--
  Это блок разметки, который использует директиву v-for для создания повторяющихся элементов.
  Для каждого объекта "city" в массиве "savedCity" будет создан элемент div.
  :key="city.id" используется для определения уникального ключа для каждого элемента, чтобы Vue мог корректно управлять их обновлением.
  -->
    <CityCard :city="city" @click="goToCityView(city)" />
    <!--
  Этот код вставляет компонент CityCard для отображения информации о городе.
  :city="city" передает объект "city" в компонент CityCard через проп "city".
  @click="goToCityView(city)" устанавливает обработчик события "click", который вызывает функцию "goToCityView(city)" при клике на компонент CityCard.
  --></div>

  <p v-if="savedCity.length === 0">
    No locations added. To start tracking a location, search in the field above.
  </p>
</template>

<script setup>
import axios from "axios";
import CityCard from "./CityCard.vue";
import { ref } from "vue";
import { useRouter } from "vue-router";
//savedCity: Это переменная, созданная с использованием Vue Composition API, используемая для хранения информации о городах. Она представляет массив городов.
const savedCity = ref([]);
const getCities = async () => {
  //getCities: Это асинхронная функция, которая служит для получения информации о городах.
  // Она начинается с проверки наличия сохраненных данных о городах в локальном хранилище браузера. Если такие данные существуют, они извлекаются и преобразуются из формата JSON в массив savedCity.value.
  if (localStorage.getItem("savedCity")) {
    savedCity.value = JSON.parse(localStorage.getItem("savedCity"));

    //requests: Это массив, в котором будут храниться асинхронные запросы к API OpenWeatherMap для каждого города.
    //axios.get(): Это метод, предоставляемый библиотекой Axios, который выполняет HTTP GET-запрос к указанному URL.
    const requests = [];
    savedCity.value.forEach((city) => {
      requests.push(
        axios.get(
          `https://api.openweathermap.org/data/2.5/weather?lat=${city.coords.lat}&lon=${city.coords.lng}&appid=7efa332cf48aeb9d2d391a51027f1a71&units=imperial`
        )
      );
    });

    //weatherData: Это переменная, в которой будут храниться данные о погоде для каждого города после выполнения всех асинхронных запросов.
    //Promise.all(): Это метод, который ожидает выполнения всех обещаний (промисов) в массиве и возвращает новый промис,
    //который разрешается массивом результатов, когда все промисы выполнены. В данном контексте, Promise.all(requests) ожидает выполнения всех асинхронных запросов в массиве requests.
    const weatherData = await Promise.all(requests);

    //weatherData.forEach(): Этот метод итерирует по каждому элементу массива weatherData.
    //value: Это объект с данными о погоде для конкретного города, полученный из выполненного запроса.
    weatherData.forEach((value, index) => {
      //savedCity.value[index].weather: Это свойство, в котором сохраняются данные о погоде для соответствующего города в массиве savedCity.value.
      savedCity.value[index].weather = value.data;
    });
  }
};

await getCities();
const router = useRouter();
// Объявляем функцию goToCityView, которая будет использоваться для навигации к странице "cityView" с параметрами.

const goToCityView = (city) => {
  // Используем метод router.push() для перехода к новому маршруту.

  router.push({
    name: "cityView",
    params: { state: city.state, city: city.city },
    query: { id: city.id, lat: city.coords.lat, lng: city.coords.lng },
  });
};
</script>
