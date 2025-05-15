<template>
    <v-row>
        <v-col cols="12" md="1" v-for="(item, index) in weatherData" :key="index">
            <v-card :text="item.day.condition.text" :subtitle="index" :title="moment(item.date).format('dddd')">
                <v-img :src="item.day.condition.icon" :width="163" />
                <v-dialog max-width="500">
                    <template v-slot:activator="{ props: activatorProps }">
                        <v-btn v-bind="activatorProps" color="surface-variant" text="Open Dialog"
                            variant="flat"></v-btn>
                    </template>

                    <template v-slot:default="{ isActive }">
                        <v-card title="Dialog">
                            <v-card-text>
                                Scott is a software engineer living in San Francisco. He enjoys working with Vue.js and
                                has a passion for building user-friendly applications. He is also an avid traveler and
                                loves to explore new places.
                            </v-card-text>

                            <v-card-actions>
                                <v-spacer></v-spacer>

                                <v-btn text="Close Dialog" @click="isActive.value = false"></v-btn>
                            </v-card-actions>
                        </v-card>
                    </template>
                </v-dialog>
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
const days = ref(7)

const fetchWeatherData = async () => {
    try {
        const response = await fetch(`http://192.168.4.28:8088/system/webdev/samplequickstart/API/forecast.json` +
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
