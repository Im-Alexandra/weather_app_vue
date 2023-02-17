<template>
  <div
    class="flex py-6 px-3 bg-weather-secondary rounded-md shadow-md cursor-pointer"
  >
    <div class="flex flex-col flex-1">
      <h2 class="text-3xl">{{ city.city }}</h2>
      <h3>{{ city.state }}</h3>
    </div>
    <div class="flex flex-col gap-2">
      <p class="text-3xl self-end">
        {{ Math.round(toCelsius(city.weather.main.temp) * 10) / 10 }}&deg;
      </p>
      <div class="flex gap-2">
        <span class="text-xs">
          H:
          {{ Math.round(toCelsius(city.weather.main.temp_max) * 10) / 10 }}&deg;
        </span>
        <span class="text-xs">
          L:
          {{ Math.round(toCelsius(city.weather.main.temp_min) * 10) / 10 }}&deg;
        </span>
      </div>
    </div>
  </div>
</template>

<script setup>
defineProps({
  city: {
    type: Object,
    default: () => {},
  },
});

const toCelsius = (farenheit) => {
  return ((farenheit - 32) * 5) / 9;
};
</script>
