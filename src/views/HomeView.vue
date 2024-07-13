<script setup>
import axios from 'axios';
import { ref } from 'vue';
import { useRouter } from 'vue-router';
import CityList from '../components/CityList.vue'
import CityCardSkeleton from '@/components/AnimatedPlaceholder/Skeleton/CityCardSkeleton.vue';

const mapboxAPIKey = "pk.eyJ1Ijoic3VuZGF5ZGF2aWQiLCJhIjoiY2x5Z3dpcHR2MGdpZDJpczV0eTI5Y212YSJ9.tZr6YrUAVsD3kXMwAm-XgQ"
const searchQuery = ref("")
const queryTimeout = ref(null)
const mapboxSearchResults = ref(null)
const searchError = ref(null)

const router = useRouter()

function getSearchResults() {
  clearTimeout(queryTimeout.value)

  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value !== "") {
      try {
        const result = await axios.get(`https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxAPIKey}&types=place`)
        mapboxSearchResults.value = result.data.features;
      } catch (error) {
        searchError.value = error
      }
      return
    }
    mapboxSearchResults.value = null
  }, 300)
}

const previewCity = (searchResult) => {
  const [city, state] = searchResult.place_name.split(",")
  router.push({
    name: 'cityView',
    params: {
      state: state.replaceAll(), city: city
    },
    query: {
      lng: searchResult.geometry.coordinates[0],
      lat: searchResult.geometry.coordinates[1],
      preview: true
    }
  })
}
</script>

<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input v-model="searchQuery" @input="getSearchResults" type="text" placeholder="Search for a city or state"
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]">
      <ul v-if="mapboxSearchResults"
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]">
        <p v-if="searchError">{{ searchError }}</p>
        <p v-if="!searchError && mapboxSearchResults.length === 0">
          No results match your query, try a different term.
        </p>
        <template v-else>
          <li v-for="searchResult in mapboxSearchResults" :key="searchResult.id" @click="previewCity(searchResult)"
            class="py-2 cursor-pointer">
            {{ searchResult.place_name }}
          </li>
        </template>
      </ul>
    </div>
    <div class="flex flex-col gap-4">
      <Suspense>
        <CityList />
        <template #fallback>
          <CityCardSkeleton />
        </template>
      </Suspense>
    </div>
  </main>
</template>
