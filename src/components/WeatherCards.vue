<template>
    <v-row>
        <v-col v-if="isLoading" cols="12" class="text-center">
            <v-progress-circular indeterminate color="primary"></v-progress-circular>
        </v-col>
        <v-col v-else-if="errorMessage" cols="12">
            <v-alert type="error">{{ errorMessage }}</v-alert>
        </v-col>
        <v-col v-else cols="12" md="1" v-for="(item, index) in formattedWeatherData" :key="index">
            <v-card :text="item.condition" :title="item.weekday">
                <v-card-subtitle>H:{{ item.maxTemp }}° L:{{ item.minTemp }}°</v-card-subtitle>
                <v-img :src="item.icon" :width="160" />
            </v-card>
        </v-col>
    </v-row>
</template>
<script setup>
import { ref, computed, inject, watch, onMounted } from 'vue'
import moment from 'moment'

const weatherData = ref([])
const days = ref(10)
const isLoading = ref(false)
const errorMessage = ref('')

// Get injected city from parent component
const city = inject('city', ref('London'))
const fetchWeatherData = async () => {
    try {
        errorMessage.value = ''
        isLoading.value = true
        const response = await fetch(`http://localhost:8088/system/webdev/samplequickstart/API/forecast.json` +
            `?q=${encodeURIComponent(city.value)}&days=${days.value}&aqi=no&alerts=no`)

        if (!response.ok) {
            throw new Error(`Location "${city.value}" could not be found. Please try another location.`)
        }

        const data = await response.json()

        if (!data.forecast || !data.forecast.forecastday) {
            throw new Error('Invalid API response format')
        }

        weatherData.value = data.forecast.forecastday
    } catch (error) {
        console.error('Error fetching weather data:', error)
        errorMessage.value = error.message || `Weather data for "${city.value}" could not be found. Please try another location.`
        weatherData.value = []
    } finally {
        isLoading.value = false
    }
}
// Watch for changes to the city reference
watch(city, (newCity) => {
    if (newCity) {
        fetchWeatherData()
    }
}, { immediate: true })

onMounted(() => {
    fetchWeatherData()
})

// Create a computed property that formats your weather data
const formattedWeatherData = computed(() => {
    return weatherData.value.map(item => {
        return {
            weekday: moment(item.date).format('dddd'),
            condition: item.day.condition.text,
            maxTemp: Math.round(item.day.maxtemp_f),
            minTemp: Math.round(item.day.mintemp_f),
            icon: item.day.condition.icon
        }
    })
})
</script>
