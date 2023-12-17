<!--Компонент для того что бы брать API DATA с сайт Open Weather API 
при помощи ключа к доступу в базу данных погоды со всего мира в реальном времени
корректировать с функциями с гогродами штатами и временем и импортировать все это в 
компонент CityView-->

<!--пишем стили для раздела показывающего погоду и корректируем данные берем из них все нужное(температуру город и т.д.)-->
<template>
  <div class="flex flex-col flex-1 items-center">
    <div
      v-if="route.query.preview"
      class="text-white p-4 bg-weather-secondary w-full text-center"
    >
      <!--      v-if="route.query.preview" показывать только если пользователь не добавил город в избранные-->
      <p>
        You are currently previewing this city, click the "+" icon to start
        tracking this city.
      </p>
    </div>
    <!--компонент для обзора погоды с помощью которого мы берем с данных который вернул API 
    потом мы показываем на экране пользователя и стлизуем их -->
    <div class="flex flex-col items-center text-white py-12">
      <!--{{ route.params.city }}  ссылка для промотра города со стилем-->
      <h1 class="text-4xl mb-2">{{ route.params.city }}</h1>
      <p class="text-sm mb-12">
        <!--здесь мы ссылаемся на запрос с данными погоды города и с помощью свойство currentTime
        показывающее текущее время и благодаря методу toLocaleDateString мы собираемся перевести его в англиский формат 
      день недели коротким, день двузначным и месяц длинным-->
        {{
          new Date(weatherData.currentTime).toLocaleDateString("en-us", {
            weekday: "short",
            day: "2-digit",
            month: "long",
          })
        }}
        {{
          //также пишем для того что бы показать время
          new Date(weatherData.currentTime).toLocaleTimeString("en-us", {
            timeStyle: "short",
          })
        }}
      </p>
      <p class="text-8xl mb-8">
        <!--метод Math.round помогает округлить цифру без дробей 
        weatherData.current.temp ссылаеммя к данным temp в реальном времени с помощью current -->
        {{ Math.round(weatherData.current.temp) }}&deg;
      </p>
      <p>
        <!--такая же функция но тут ощущение погоды-->
        Feels like
        {{ Math.round(weatherData.current.feels_like) }}&deg;
      </p>
      <p class="capitalize">
        <!--текущее описание погоды -->
        {{ weatherData.current.weather[0].description }}
      </p>
      <!--в качестве привязания иконок мы используем иконки которые уже есть на сайте open weather и ссылаемся на текущию иконку погоды -->
      <img
        class="w-[150px] h-auto"
        :src="`http://openweathermap.org/img/wn/${weatherData.current.weather[0].icon}@2x.png`"
        alt=""
      />
    </div>

    <hr class="border-white border-opacity-10 border w-full" />

    <!--раздел для показа почасового показа погоды-->
    <div class="max-w-screen-md w-full py-12">
      <div class="mx-8 text-white">
        <h2 class="mb-4">Hourly Weather</h2>
        <div class="flex gap-10 overflow-x-scroll">
          <!--для того что бы показывать погоду дял каждого часа у нас API данных есть массив данных с именем hourly 
          благодаря которому мы можем сделать иттерацию с помощью директивы v-for по массивам что бы показать погоду по каждому часу   -->
          <div
            v-for="hourData in weatherData.hourly"
            :key="hourData.dt"
            class="flex flex-col gap-4 items-center"
          >
            <!--раздел для того что бы показать время почасавой погоды так же как и для в верхнем разделе-->
            <p class="whitespace-nowrap text-md">
              {{
                new Date(hourData.currentTime).toLocaleTimeString("en-us", {
                  hour: "numeric",
                })
              }}
            </p>
            <!--такой же раздел для показа иконок для каждой иттераций с погодой -->
            <img
              class="w-auto h-[50px] object-cover"
              :src="`http://openweathermap.org/img/wn/${hourData.weather[0].icon}@2x.png`"
              alt=""
            />
            <!--раздел для показа погоды для каждой иттераций почасовой погоды-->
            <p class="text-xl">{{ Math.round(hourData.temp) }}&deg</p>
          </div>
        </div>
      </div>
    </div>

    <hr class="border-white border-opacity-10 border w-full" />

    <!--раздел для показа погоды каждого дня недели-->
    <div class="msx-w-screen w-full py-12">
      <div class="mx-8 text-white">
        <h2 class="mb-4">7 Day Forecast</h2>
        <div
          class="flex items-center"
          v-for="day in weatherData.daily"
          :key="day.dt"
        >
          <p class="flex-1">
            {{
              new Date(day.dt * 1000).toLocaleDateString("en-us", {
                weekday: "long",
              })
            }}
          </p>
          <img
            class="w-[50px] h-[50px] object-cover"
            :src="`http://openweathermap.org/img/wn/${day.weather[0].icon}@2x.png`"
            alt=""
          />
          <!--раздел для показа погоды максимального и минимальной темпиратуры с таким же методом round-->
          <div class="flex gap-2 flex-1 justify-end">
            <p>H:{{ Math.round(day.temp.max) }}</p>
            <p>L:{{ Math.round(day.temp.min) }}</p>
          </div>
        </div>
      </div>
    </div>

    <div
      class="flex items-center gap-2 py-12 text-white cursor-pointer duration-150 hover:text-red-500"
      @click="removeCity"
    >
      <!--компонент для удаления избранных городов-->
      <i class="fa-solid fa-trash"> Remove City </i>
    </div>
  </div>
</template>

<script setup>
import axios from "axios";
import { useRoute, useRouter } from "vue-router";

const route = useRoute();
//Асинхронная функция в котором при помощи GET запроса аксиос мы получаем данные погод всех городов, обязательно нужно API ключ к доступу к данным
//что бы найти определенный город мы ссылаемся на координаты городов которые корректировали в компоненте HomeView
const getWeatherData = async () => {
  try {
    const weatherData = await axios.get(
      `https://api.openweathermap.org/data/2.5/onecall?lat=${route.query.lat}&lon=${route.query.lng}&exclude={part}&appid=7efa332cf48aeb9d2d391a51027f1a71&units=imperial`
    );

    // Здесь вычисляется разница во времени между текущим местным временем и временем UTC.
    const localOffset = new Date().getTimezoneOffset() * 60000;
    //
    const utc = weatherData.data.current.dt * 1000 + localOffset;
    weatherData.data.currentTime =
      utc + 1000 * weatherData.data.timezone_offset;

    //Выполняется итерация по каждому часовому прогнозу в массиве hourly в объекте
    weatherData.data.hourly.forEach((hour) => {
      const utc = hour.dt * 1000 + localOffset;
      hour.currentTime = utc + 1000 * weatherData.data.timezone_offset;
    });

    return weatherData.data;
  } catch (err) {
    console.log(err);
  }
};
//здесь мы вызываем функцию переименуя ее в асинхронную что бы импортировать в компонент CityView
//и что бы импорт данных асинхронной функций прошел успешно мы воспользуемся свойством во VUE под тэгом <Suspense>
//в котором
const weatherData = await getWeatherData();

const router = useRouter();
//removeCity(): Это объявление функции removeCity, которая будет выполнять действия по удалению города и переходу на домашнюю страницу.
const removeCity = () => {
  //cities: Это переменная, в которой сохраняются данные о городах, извлеченные из локального хранилища.
  //JSON.parse(): Это метод JavaScript для преобразования строки JSON в объект JavaScript. В данном случае, используется для извлечения данных о городах из локального хранилища браузера.
  //localStorage.getItem("savedCity"): Это метод getItem объекта localStorage, который получает значение по ключу "savedCity" из локального хранилища браузера. Этот ключ, вероятно, используется для хранения списка городов.
  const cities = JSON.parse(localStorage.getItem("savedCity"));
  //.filter(): Это метод массива, который выполняет фильтрацию массива в соответствии с заданным условием. В данном случае, он фильтрует массив городов, оставляя только те, чей идентификатор (id) не совпадает с переданным id из текущего маршрута
  //передаем маршрут и уникальный идентификатор через uid
  const updatedCities = cities.filter((city) => city.id !== route.query.id);
  //updatedCities: Это переменная, в которой сохраняются обновленные данные о городах после фильтрации.
  localStorage.setItem("savedCity", JSON.stringify(updatedCities));
  router.push({
    name: "home",
  });
};
</script>
