<template>
    <div v-for="city in savedCities" :key="city.id">
        <CityCard :city="city" @click="gotoCityView(city)" />
    </div>

    <p v-if="savedCities.length === 0">
        No location added. To start tracking a location, search in the field above.
    </p>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'
import CityCard from './CityCard.vue'
import { useRouter } from 'vue-router';

const savedCities = ref([])

const getCities = async () => {
    if (localStorage.getItem("savedCities")) {
        savedCities.value = JSON.parse(localStorage.getItem("savedCities"));

        const requests = []
        savedCities.value.forEach((city) => {
            requests.push(axios.get(`https://api.openweathermap.org/data/2.5/weather?lat=${city.coords.lat}&lon=${city.coords.lng}&appid=214f2db52db53ac401a08723ccaad17e&units=imperial`))
        });

        const weatherData = await Promise.all(requests);

        await new Promise((res) => setTimeout(res, 1000))

        weatherData.forEach((value, index) => {
            savedCities.value[index].weather = value.data
        })
    }
}

await getCities()

const router = useRouter()
function gotoCityView(city) {
    router.push({
        name: "cityView",
        params: { state: city.state, city: city.city },
        query: { id: city.id, lat: city.coords.lat, lng: city.coords.lng }
    })
}

</script>