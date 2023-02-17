<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
        type="text"
        v-model="searchQuery"
        @input="getSearchResults"
        placeholder="Search for a city or state"
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]"
      />
      <ul
        v-if="mapboxSearchResults"
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
      >
        <p v-if="searchError">Something went wrong, try again later.</p>
        <p v-if="!searchError && mapboxSearchResults.length === 0">
          No results match this search, try a different term.
        </p>
        <template v-else>
          <li
            v-for="searchResult in mapboxSearchResults"
            :key="searchResult.id"
            class="py-2 cursor-pointer"
            @click="previewCity(searchResult)"
          >
            {{ searchResult.place_name }}
          </li>
        </template>
      </ul>
    </div>
    <div class="flex flex-col gap-4">
      <!-- using suspense here because CityList is asynchronous -->
      <Suspense>
        <CityList />
        <template #fallback><CityCardSkeleton /></template>
      </Suspense>
    </div>
  </main>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";
import { useRouter } from "vue-router";
import CityList from "../components/CityList.vue";
import CityCardSkeleton from "../components/CityCardSkeleton.vue";

const searchQuery = ref("");
const queryTimeout = ref(null);
const mabboxAPIKey =
  "pk.eyJ1IjoiaW0tYWxleGFuZHJhIiwiYSI6ImNsZTRjbjMyZzAxdDgzb3M2dXc2bHljeHIifQ.d4JzcIvsy5QgrXXmVOm55w";
const mapboxSearchResults = ref(null);
const searchError = ref(null);
const router = useRouter();

const previewCity = (searchResult) => {
  const [city, state, country] = searchResult.place_name.split(",");
  router.push({
    name: "CityView",
    params: {
      state: state.replaceAll(" ", ""),
      city: city.replaceAll(" ", ""),
    },
    query: {
      lat: searchResult.geometry.coordinates[1],
      lng: searchResult.geometry.coordinates[0],
      preview: true,
    },
  });
};

const getSearchResults = () => {
  clearTimeout(queryTimeout.value);
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value !== "") {
      try {
        const result = await axios.get(
          `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mabboxAPIKey}&types=place`
        );
        mapboxSearchResults.value = result.data.features;
      } catch (err) {
        searchError.value = true;
      }

      return;
    }
    mapboxSearchResults.value = null;
  }, 300);
};
</script>
