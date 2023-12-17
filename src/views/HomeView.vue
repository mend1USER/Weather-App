<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <!-- метод двустороннего связывания что бы обновлять значение -->
      <input
        type="text"
        v-model="searchQuery"
        @input="getSearchResults"
        placeholder="Search for a city or state"
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]"
      />
      <ul
        v-if="mapboxSearchResults"
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-2 top[66px]"
      >
        <!--здесь мы пишем валидацию форм для предотврощения ошибок
      если будет что то с API даннымми-->
        <p v-if="searchError">Sorry, something went wrong, please try again.</p>
        <p v-if="!serverError && mapboxSearchResults.length === 0">
          No results match your query, try a different term.
        </p>
        <template v-else>
          <!-- v-for для создания списка результатов поиска 
          на основе данных из массива mapboxSearchResults -->
          <li
            class="py-2 cursor-pointer"
            v-for="searchResult in mapboxSearchResults"
            :key="searchResult.id"
            @click="previewCity(searchResult)"
          >
            {{ searchResult.place_name }}
          </li>
        </template>
      </ul>
    </div>
    <div class="flex flex-col gap-4">
      <!--тот же для асинхронной функций-->
      <Suspense>
        <!--импортируем ситилист для того оно показывало на главном экарне-->
        <CityList />
        <template #fallback>
          <p>Loading...</p>
        </template>
      </Suspense>
    </div>
  </main>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";
import { useRouter } from "vue-router";
import CityList from "../components/CityList.vue";

const router = useRouter();

//это функция в котором сначала при клике пользователя
//на кнопку превью сити с аргументом серч сити после ктотрого
//в консоль отправляються данные с городм который ищет пользователь
//и в качестве тела функций мы сперва  раздели строки в обьекте серчрезалт по запятой
//потом через маршрутизатор пуш мы переходим на новый маршрут с именем CityView
//и передаем какие параметры нужны для маршрута

const previewCity = (searchResult) => {
  console.log(searchResult);
  //мы равняем параметры стэйт и сити к параметрам запроса а именно к place_name где находять город и штат запроса и разделяем их с помощью свойство сплит
  const [city, state] = searchResult.place_name.split(",");
  router.push({
    name: "cityView",
    // state.replaceAll(" ", "") используется для удаления пробелов из state.
    params: { state: state.replaceAll(" ", ""), city: city },
    //обьект запроса куери в котором есть параметры которые будут для дополнительной передачи данных
    //данный объект запроса определяет три параметра: lat и lng для координат местоположения
    //которые находятся внутри массива данных geometry нутри них coordinates и я ссылался на них с помощью length
    //и preview для указания режима предварительного просмотра
    query: {
      lat: searchResult.geometry.coordinates[1],
      lng: searchResult.geometry.coordinates[0],
      preview: true,
    },
  });
};

//переменная в которой храниться ключ к Mapbox Geocoding API с помощью которого мы получаем местоположение указанное в пойсковом запросе
const mapboxAPIKey =
  "pk.eyJ1Ijoiam9obmtvbWFybmlja2kiLCJhIjoiY2t5NjFzODZvMHJkaDJ1bWx6OGVieGxreSJ9.IpojdT3U3NENknF6_WhR2Q";
const queryTimeout = ref(null);
//переменная серчкуери мы присваиваем к реактивной переменной реф и делаем значение пустой строкой
//потому что эта переменная это пойсковой запрос и с помощью свойстов в-модел мы делаем
//двусторннию связку которая в реальном времени будет обновлять значение переменной серчкуери
const searchQuery = ref("");
//Это реактивная переменная, которая будет хранить сообщение об ошибке
const searchError = ref(null);
//mapboxSearchResults - Это реактивная переменная, которая будет хранить результаты поиска городов и штатов.
const mapboxSearchResults = ref(null);

//функция которая будет находить города с помощью mapboxAPI
const getSearchResults = () => {
  //clearTimeout(queryTimeout.value) - этот код используется для отмены предыдущего таймера,
  //который был установлен с помощью setTimeout в функции getSearchResults.
  clearTimeout(queryTimeout.value);
  //это таймер с задержкой 300 милесекунд для того чтобы запрос делался псоле истечения времени
  //что бы снижать нагрузку на сервер
  queryTimeout.value = setTimeout(async () => {
    //если пойсковой запрос пустой оно должно выполнять эту функцию...
    if (searchQuery.value !== "") {
      try {
        //await - ключевое слово, которое указывает, что выполнение кода должно ожидать завершения асинхронной операции,
        // прежде чем продолжить выполнение следующего кода.
        // В данном случае, мы ожидаем выполнения запроса, прежде чем присваивать результат переменной result.
        //axios.get(url) - это вызов функции get библиотеки Axios, который выполняет GET-запрос по указанному URL.
        //URL запроса формируется следующим образом:
        //https://api.mapbox.com/geocoding/v5/mapbox.places/ - это базовый URL для Mapbox Geocoding API.
        //${searchQuery.value} - это значение, введенное пользователем в поле поиска, которое добавляется к URL. Это значение используется для выполнения поиска по городам или штатам.
        //.json - указывает, что мы ожидаем получить ответ в формате JSON.
        //и последняя чать запроса это параматры запросы с помощью ключа к Mapbox Geocoding API благодаря которой мы получаем местопопложение указанное в пойсковом запросу в формате JSON
        const result = await axios.get(
          `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxAPIKey}&types=place`
        );
        //mapboxSearchResults.value = result.data.features - Здесь мы извлекаем из результатов запроса объект features,
        //который содержит информацию о найденных местоположениях (городах и штатах),
        // и сохраняем его в реактивной переменной mapboxSearchResults.
        // Теперь эта переменная содержит результаты поиска и может быть использована для отображения на странице.
        mapboxSearchResults.value = result.data.features;
        // Этот код выводит результаты поиска в консоль для отладки. Это поможет вам убедиться,
        // что данные были получены и обработаны правильно.
        console.log(mapboxSearchResults.value);
      } catch {
        // блоке catch обрабатывается ошибка, если при выполнении запроса произошла ошибка.
        //В данном случае, если произошла ошибка, устанавливается значение searchError.value в true.
        //Это позволит вам отобразить сообщение об ошибке на странице.
        searchError.value = true;
      }

      return;
    }
    //если нету никаких результатов по городам штатам то просто возварщяем нулл
    mapboxSearchResults.value = null;
  }, 300);
};
</script>

<style scoped></style>
