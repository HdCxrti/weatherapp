<template>
    <v-row>
        <v-col cols="12" md="1" v-for="(item, index) in weatherData" :key="index">
            <v-card :text="item.day.condition.text" :title="moment(item.date).format('dddd')">
                <v-card-subtitle>H:{{ Math.round(item.day.maxtemp_f) }}° L:{{ Math.round(item.day.mintemp_f)
                }}°</v-card-subtitle>
                <v-img :src="item.day.condition.icon" :width="160" />
            </v-card>
        </v-col>
    </v-row>
</template>
<script setup>
import moment from 'moment'
import { ref, onMounted } from 'vue'
const momentString = ref('Click for Moment')
const weatherData = ref([])
const city = ref('London')
const days = ref(10)

const fetchWeatherData = async () => {
    try {
        const response = await fetch(`http://localhost:8088/system/webdev/samplequickstart/API/forecast.json` +
            `?q=${encodeURIComponent(city.value)}&days=${days.value}&aqi=no&alerts=no`)

        if (!response.ok) {
            throw new Error('Network response was not ok')
        }
        const data = await response.json()

        weatherData.value = data.forecast.forecastday
    } catch (error) {
        console.error('Error fetching weather data:', error)

    }
}

onMounted(() => {
    fetchWeatherData()
})
</script>
