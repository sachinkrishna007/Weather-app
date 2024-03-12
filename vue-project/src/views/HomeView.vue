<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
        type="text"
        @input="getSearchResults"
        v-model="searchQuery"
        placeholder="Search City "
        class="py-2 px-1 w-full bg-transparent border-b focus:outline-none focus:border-weather-secondary focus:shadow-sm"
      />
      <ul
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
        v-if="mapBoxResult"
      >
        <p v-if="searchError">Sooory,Try again</p>
        <p v-if="!serverError && mapBoxResult.length === 0">No results found</p>
        <template v-else>
          <li
            v-for="searchResult in mapBoxResult"
            :key="searchResult.id"
            class="py-2 cursor-pointer"
            @click="previewCity(searchResult)"
          >
            {{ searchResult.place_name }}
          </li>
        </template>
      </ul>
    </div>
  </main>
</template>

<script setup>
import axios from "axios";
import { ref } from "vue";
import { useRouter } from "vue-router";
const router = useRouter();
const previewCity = (searchResult) => {
  const [city, state] = searchResult.place_name.split(",");
  router.push({
    name: "CityView",
    params: { state: state.replaceAll(" ", ""), city: city },
    query: {
      lat: searchResult.geometry.coordinates[1],
      lnt: searchResult.geometry.coordinates[0],
      preview: true,
    },
  });
};

const mapboxAPIkey =
  "pk.eyJ1Ijoic2FjaGlua3Jpc2huYTE5OTgiLCJhIjoiY2x0bWh4dGJyMWpvMjJrcmhiNHNuZTB6YiJ9.1NRopXy_FQvCIBMpUHlyCw";

const searchQuery = ref("");
const queryTimeout = ref("");
const mapBoxResult = ref("");
const searchError = ref("");
const getSearchResults = () => {
  clearTimeout(queryTimeout.value);
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value !== "") {
      try {
        const result = await axios.get(
          `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxAPIkey}&types=place`
        );
        mapBoxResult.value = result.data.features;
      } catch {
        searchError.value = true;
      }

      return;
    }
    mapBoxResult.value = null;
  }, 300);
};
</script>
