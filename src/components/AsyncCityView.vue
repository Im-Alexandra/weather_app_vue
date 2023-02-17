<template>
  <div class="flex flex-col flex-1 items-center">
    <!-- Banner-->
    <div
      v-if="route.query.preview"
      class="text-white p-4 bg-weather-secondary w-full text-center"
    >
      <p>
        You are currently previewing this city, click the "+" icon to start
        tracking this city.
      </p>
    </div>
    <!-- Weather overview -->
    <div class="flex flex-col items-center text-white py-12">
      <h1 class="text-4xl mb-2">{{ route.params.city }}</h1>
      <p class="text-sm mb-12">
        {{
          new Date(weatherData.currentTime).toLocaleDateString("en-uk", {
            weekday: "short",
            day: "2-digit",
            month: "long",
          })
        }}
        {{
          new Date(weatherData.currentTime).toLocaleTimeString("en-uk", {
            timeStyle: "short",
          })
        }}
      </p>
      <p class="text-8xl mb-8">
        {{ Math.round(toCelsius(weatherData.current.temp) * 10) / 10 }}&deg;
      </p>

      <p>
        Feels like
        {{
          Math.round(toCelsius(weatherData.current.feels_like) * 10) / 10
        }}&deg;
      </p>
      <p class="capitalize">
        {{ weatherData.current.weather[0].description }}
      </p>
      <img
        class="w-[150px] h-auto"
        :src="`http://openweathermap.org/img/wn/${weatherData.current.weather[0].icon}@2x.png`"
        alt="current weather"
      />
    </div>
    <!-- Separation -->
    <hr class="border-white border-opacity-10 border w-full" />
    <!-- Hourly overview -->
    <div class="max-w-screen-md w-full py-12">
      <div class="mx-8 text-white">
        <h2 class="mb-4">Hourly Weather</h2>
        <div class="flex gap-10 overflow-x-scroll pb-4">
          <div
            v-for="hourData in weatherData.hourly"
            :key="hourData.dt"
            class="flex flex-col gap-4 items-center"
          >
            <p class="whitespace-nowrap text-md">
              {{
                new Date(hourData.currentTime).toLocaleTimeString("en-uk", {
                  hour: "numeric",
                })
              }}:00
            </p>
            <img
              class="w-auto h-[50px] object-cover"
              :src="`http://openweathermap.org/img/wn/${hourData.weather[0].icon}@2x.png`"
              alt="hourly weather"
            />
            <p class="text-xl">
              {{ Math.round(toCelsius(hourData.temp) * 10) / 10 }}&deg;
            </p>
          </div>
        </div>
      </div>
    </div>
    <!-- Separation -->
    <hr class="border-white border-opacity-10 border w-full" />
    <!-- Weekly overview -->
    <div class="max-w-screen-md w-full py-12">
      <div class="mx-8 text-white">
        <h2 class="mb-4">7 Day Forecast</h2>
        <div
          v-for="day in weatherData.daily"
          :key="day.dt"
          class="flex items-center"
        >
          <p class="flex-1">
            {{
              new Date(day.dt * 1000).toLocaleDateString("en-uk", {
                weekday: "long",
              })
            }}
          </p>
          <img
            class="w-[50px] h-[50px] object-cover"
            :src="`http://openweathermap.org/img/wn/${day.weather[0].icon}@2x.png`"
            alt="daily weather"
          />
          <div class="flex gap-2 flex-1 justify-end">
            <p>H: {{ Math.round(toCelsius(day.temp.max) * 10) / 10 }}&deg;</p>
            <p>L: {{ Math.round(toCelsius(day.temp.min) * 10) / 10 }}&deg;</p>
          </div>
        </div>
      </div>
    </div>
    <!-- Remove from local storage -->
    <div
      @click="removeCity"
      class="flex items-center gap-2 py-12 text-white cursor-pointer duration-150 hover:text-red-500"
    >
      <i class="fa-solid fa-trash"></i>
      <p>Remove city</p>
    </div>
  </div>
</template>

<script setup>
import axios from "axios";
import { useRoute, useRouter } from "vue-router";

const route = useRoute();
const router = useRouter();

const toCelsius = (farenheit) => {
  return ((farenheit - 32) * 5) / 9;
};

const getWeatherData = async () => {
  try {
    const weatherData = await axios.get(
      `https://api.openweathermap.org/data/2.5/onecall?lat=${route.query.lat}&lon=${route.query.lng}&exclude={part}&appid=7efa332cf48aeb9d2d391a51027f1a71&units=imperial`
    );

    //call current date & time
    const localOffset = new Date().getTimezoneOffset() * 60000;
    const utc = weatherData.data.current.dt * 1000 + localOffset;
    weatherData.data.currentTime =
      utc + 1000 * weatherData.data.timezone_offset;

    //call hourly weather offset
    weatherData.data.hourly.forEach((hour) => {
      const utc = hour.dt * 1000 + localOffset;
      hour.currentTime = utc + 1000 * weatherData.data.timezone_offset;
    });

    //Flicker delay to display skeleton animation
    await new Promise((res) => setTimeout(res, 1000));

    return weatherData.data;
  } catch (err) {
    console.log(err);
  }
};

const weatherData = await getWeatherData();

const removeCity = () => {
  const cities = JSON.parse(localStorage.getItem("savedCities"));
  const updatedCities = cities.filter((city) => {
    return city.id !== route.query.id;
  });
  localStorage.setItem("savedCities", JSON.stringify(updatedCities));
  router.push({ name: "Home" });
};
</script>

<style scoped>
::-webkit-scrollbar {
  height: 8px;
  width: 4px;
  background: transparent;
}
::-webkit-scrollbar-thumb:horizontal {
  background: #48484a;
  border-radius: 10px;
}
</style>
