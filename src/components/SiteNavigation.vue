<template>
    <header class="sticky top-0 bg-weather-primary shadow-lg">
        <nav class="container flex flex-col sm:flex-row items-center gap-4 text-white py-6">
            <RouterLink :to="{ name: 'home' }">
                <div class="flex items-center gap-3">
                    <i class="fa-solid fa-sun text-2xl"></i>
                    <p class="text-2xl">Local Weather</p>
                </div>
            </RouterLink>
            <div class="flex gap-3 flex-1 justify-end">
                <i @click="toggleModal"
                    class="fa-solid fa-circle-info text-xl hover:text-weather-secondary duration-150 cursor-pointer"></i>
                <i class="fa-solid fa-plus text-xl hover:text-weather-secondary duration-150 cursor-pointer"
                    @click="addCity" v-if="route.query.preview"></i>
            </div>

            <BaseModal @close-modal="toggleModal" :modalActive="modalActive">
                <div class="text-black">
                    <h1 class="text-2xl mb-1">About:</h1>
                    <p class="mb-4">The local Weather allows you to track the current and future weather of cities of
                        your choosing</p>

                    <h1 class="text-2xl">How it works:</h1>
                    <ol class="list-decimal list-inside mb-4">
                        <li>Search for your city by entering the name into the search bar</li>
                        <li>Search for your city by entering the name into the search bar</li>
                        <li>Search for your city by entering the name into the search bar</li>
                        <li>Search for your city by entering the name into the search bar</li>
                    </ol>
                </div>
            </BaseModal>
        </nav>
    </header>
</template>

<script setup>
import { RouterLink, useRoute, useRouter } from 'vue-router';
import BaseModal from './BaseModal.vue'
import { ref } from 'vue';
import { uid } from 'uid';

const modalActive = ref(null)
const saveCities = ref([])
const route = useRoute()
const router = useRouter()

function toggleModal() {
    modalActive.value = !modalActive.value
}

function addCity() {
    const savedCitiesFromStorage = localStorage.getItem('savedCities')
    if (savedCitiesFromStorage) {
        try {
            const parsedCities = JSON.parse(savedCitiesFromStorage)
            if (Array.isArray(parsedCities)) {
                saveCities.value = parsedCities
            } else {
                saveCities.value = []
            }
        } catch (error) {
            console.error('Error parsing saved cities:', error)
            saveCities.value = []
        }
    }

    const locationObj = {
        id: uid(),
        state: route.params.state,
        city: route.params.city,
        coords: {
            lat: route.query.lat,
            lng: route.query.lng
        }
    }

    saveCities.value.push(locationObj)
    localStorage.setItem("savedCities", JSON.stringify(saveCities.value))

    let query = Object.assign({}, route.query)
    delete query.preview;
    query.id = locationObj.id
    router.replace({ query })
}
</script>