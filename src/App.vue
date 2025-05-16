<template>
  <v-app id="inspire" :class="weatherBackgroundClass" :theme="theme">
    <!-- Add a background container -->
    <div class="bg-container"></div>

    <!-- Rest of your app content -->
    <v-app-bar flat>
      <v-container class="mx-auto d-flex align-center justify-center">
        <v-responsive class="d-flex align-center">
          <div class="d-flex align-center">
            <v-text-field v-model="searchCity" density="compact" label="Search city" rounded="lg" variant="solo-filled"
              flat hide-details single-line class="me-2" @keyup.enter="searchWeather"></v-text-field>
            <v-btn color="primary" icon @click="searchWeather" :loading="isSearching">
              <v-icon>mdi-magnify</v-icon>
            </v-btn>

            <!-- Add dark mode toggle -->
            <v-btn icon class="ms-2" @click="toggleTheme">
              <v-icon>{{ theme === 'light' ? 'mdi-weather-night' : 'mdi-weather-sunny' }}</v-icon>
            </v-btn>
          </div>
        </v-responsive>
      </v-container>
    </v-app-bar>

    <v-main>
      <v-container>
        <!-- Current city display with loading indicator -->
        <v-row class="mb-4">
          <v-col cols="12" class="text-center">
            <v-progress-circular v-if="locationLoading" indeterminate color="primary"
              class="mb-2"></v-progress-circular>
            <h1 class="text-h3 font-weight-bold text-white text-shadow">
              {{ currentCity }}
            </h1>
          </v-col>
        </v-row>

        <!-- Favorite Cities Expansion Panel -->
        <v-row>
          <v-col cols="12">
            <v-expansion-panels>
              <v-expansion-panel>
                <v-expansion-panel-title>
                  <div class="d-flex align-center">
                    <v-icon class="me-2">mdi-heart</v-icon>
                    <span>Favorite Cities</span>
                    <v-chip class="ms-2" size="small" color="primary" variant="outlined">{{ favoriteCities.length
                    }}</v-chip>
                  </div>
                </v-expansion-panel-title>
                <v-expansion-panel-text>
                  <div v-if="favoriteCities.length === 0" class="text-center py-3">
                    <p class="text-medium-emphasis">No favorite cities yet. Add some!</p>
                  </div>
                  <v-chip-group v-else>
                    <v-chip v-for="(city, index) in favoriteCities" :key="index" color="primary"
                      @click="selectFavoriteCity(city)" closable @click:close="removeFavorite(index)">
                      {{ city }}
                    </v-chip>
                  </v-chip-group>
                  <v-btn v-if="!isCityFavorite" size="small" prepend-icon="mdi-heart" @click="addToFavorites"
                    color="error" :disabled="!currentCity" class="mt-3">
                    Add {{ currentCity }} to favorites
                  </v-btn>
                </v-expansion-panel-text>
              </v-expansion-panel>
            </v-expansion-panels>
          </v-col>
        </v-row>

        <!-- Weather content -->
        <router-view :city="currentCity" :key="searchKey" />
      </v-container>
    </v-main>
  </v-app>
</template>

<script setup>
import { ref, provide, computed, onMounted, watch } from 'vue'

const links = [
  'Dashboard',
  'Messages',
  'Profile',
  'Updates',
]

const searchCity = ref('')
const currentCity = ref('Hedgesville') // Default city if geolocation fails
const isSearching = ref(false)
const searchKey = ref(0)
const favoriteCities = ref([])
const currentWeather = ref('')
const locationLoading = ref(false)

// Theme handling
const theme = ref('light')

// Initialize theme based on system preference
onMounted(() => {
  // Check if user prefers dark mode
  const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches
  theme.value = prefersDark ? 'dark' : 'light'

  // Listen for changes in system preference
  window.matchMedia('(prefers-color-scheme: dark)').addEventListener('change', event => {
    theme.value = event.matches ? 'dark' : 'light'
  })
})

// Function to toggle between light and dark themes
const toggleTheme = () => {
  theme.value = theme.value === 'light' ? 'dark' : 'light'
}

// Check if current city is already in favorites
const isCityFavorite = computed(() => {
  return favoriteCities.value.includes(currentCity.value)
})

// Make city and current weather available to child components
provide('city', currentCity)
provide('currentWeather', currentWeather)
provide('theme', theme) // Provide theme to child components

// Weather background class based on currentCity's weather
const weatherBackgroundClass = computed(() => {
  if (!currentWeather.value) return 'bg-default'

  const weather = currentWeather.value.toLowerCase()
  if (weather.includes('rain') || weather.includes('drizzle') || weather.includes('shower')) {
    return 'bg-rain'
  } else if (weather.includes('snow')) {
    return 'bg-snow'
  } else if (weather.includes('cloud') || weather.includes('overcast')) {
    return 'bg-cloudy'
  } else if (weather.includes('sun') || weather.includes('clear')) {
    return 'bg-sunny'
  } else if (weather.includes('thunder') || weather.includes('storm')) {
    return 'bg-storm'
  } else if (weather.includes('fog') || weather.includes('mist')) {
    return 'bg-fog'
  } else {
    return 'bg-default'
  }
})

onMounted(() => {
  // Load favorites from local storage
  const savedFavorites = localStorage.getItem('favoriteCities')
  if (savedFavorites) {
    favoriteCities.value = JSON.parse(savedFavorites)
  }

  // Try to get user's location first
  locationLoading.value = true

  // Check if geolocation is available
  if ("geolocation" in navigator) {
    navigator.geolocation.getCurrentPosition(
      // Success callback
      position => {
        const { latitude, longitude } = position.coords
        console.log("Geolocation successful:", latitude, longitude)

        // Use reverse geocoding to get city name from coordinates
        reverseGeocode(latitude, longitude)
      },
      // Error callback
      error => {
        console.warn("Geolocation error:", error.message)
        // Use default city if geolocation fails
        locationLoading.value = false
        currentCity.value = 'Hedgesville'
        searchKey.value++ // Force weather update
      },
      // Options
      {
        timeout: 10000,  // 10 second timeout
        maximumAge: 60000 // Use cached position if less than 1 minute old
      }
    )
  } else {
    // Browser doesn't support geolocation
    console.log("Geolocation not supported, using default city")
    locationLoading.value = false
    currentCity.value = 'Hedgesville'
    searchKey.value++ // Force weather update
  }
})

const reverseGeocode = async (lat, lon) => {
  try {
    console.log("Reverse geocoding for:", lat, lon)

    // Use WeatherAPI.com for geocoding
    const response = await fetch(
      `https://api.weatherapi.com/v1/search.json?key=fbd3658462f64cea8f0154659251605&q=${lat},${lon}`,
      {
        headers: {
          'Accept': 'application/json'
        }
      }
    )

    if (response.ok) {
      const data = await response.json()
      console.log("WeatherAPI.com search response:", data)

      if (data && data.length > 0) {
        // WeatherAPI.com will return the closest locations to the coordinates
        // The first result is typically the most accurate
        const location = data[0]

        // The name field contains the locality name
        let cityName = location.name

        // If we have region info, add it for better clarity
        if (location.region) {
          cityName += `, ${location.region}`
        }

        console.log("Located city:", cityName)
        currentCity.value = cityName
        searchKey.value++
        return
      }
    } else {
      console.error("WeatherAPI.com error:", await response.text())
    }

    // Fallback to default city if WeatherAPI.com fails
    console.log("Reverse geocoding failed, using default city")
    currentCity.value = 'Hedgesville'
    searchKey.value++


  } catch (error) {
    console.error("Error in reverse geocoding:", error)
    // Use default city
    currentCity.value = 'Hedgesville'
    searchKey.value++
  } finally {
    isSearching.value = false
    locationLoading.value = false
  }
}

const searchWeather = () => {
  if (!searchCity.value.trim()) return

  isSearching.value = true
  currentCity.value = searchCity.value.trim()
  currentWeather.value = '' // Reset weather when changing city

  // Changing the key will force the child component to re-render
  searchKey.value++

  setTimeout(() => {
    isSearching.value = false
  }, 1000)
}

const addToFavorites = () => {
  if (currentCity.value && !favoriteCities.value.includes(currentCity.value)) {
    favoriteCities.value.push(currentCity.value)
    // Save to local storage
    localStorage.setItem('favoriteCities', JSON.stringify(favoriteCities.value))
  }
}

const removeFavorite = (index) => {
  favoriteCities.value.splice(index, 1)
  // Update local storage
  localStorage.setItem('favoriteCities', JSON.stringify(favoriteCities.value))
}

const selectFavoriteCity = (city) => {
  currentCity.value = city
  searchKey.value++
}
</script>

<style>
/* Default background */
.bg-default {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%) !important;
}

.bg-rain {
  background-color: #3a7bd5 !important;
  background-image: url('@/assets/rain.avif') !important;
  /* Note the quotes */
  background-size: cover !important;
  background-position: center !important;
  background-attachment: fixed !important;
}

.bg-snow {
  background-color: #e6e9f0 !important;
  background-image: url('@/assets/snow.avif') !important;
  background-size: cover !important;
  background-position: center !important;
  background-attachment: fixed !important;
}

.bg-cloudy {
  background-color: #8e9eab !important;
  background-image: url('@/assets/cloudy.avif') !important;
  background-size: cover !important;
  background-position: center !important;
  background-attachment: fixed !important;
}

.bg-sunny {
  background-color: #f8b500 !important;
  background-image: url('@/assets/sunny.avif') !important;
  background-size: cover !important;
  background-position: center !important;
  background-attachment: fixed !important;
}

.bg-storm {
  background-color: #373b44 !important;
  background-image: url('@/assets/storm.avif') !important;
  background-size: cover !important;
  background-position: center !important;
  background-attachment: fixed !important;
}

.bg-fog {
  background-color: #b8d3d3 !important;
  background-image: url('@/assets/fog.avif') !important;
  background-size: cover !important;
  background-position: center !important;
  background-attachment: fixed !important;
}

/* Text shadow for better readability over background images */
.text-shadow {
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.5) !important;
}

/* Add this to create a fixed background container */
.bg-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: -1;
  background-size: cover !important;
  background-position: center !important;
  background-repeat: no-repeat !important;
}

/* Update your styles to use online images */
.bg-storm .bg-container {
  background-color: #373b44;
  background-image:
    linear-gradient(to bottom, rgba(0, 0, 0, 0.5), rgba(37, 38, 43, 0.7)),
    url('https://images.unsplash.com/photo-1605727216801-e27ce1d0cc28?q=80&w=2000&auto=format&fit=crop');
  background-size: cover !important;
  background-position: center !important;
}

.bg-rain {
  background-color: #3a7bd5;
  background-image:
    linear-gradient(to bottom, rgba(0, 0, 0, 0.4), rgba(58, 123, 213, 0.6)),
    url('https://images.unsplash.com/photo-1519692933481-e162a57d6721?q=80&w=2000&auto=format&fit=crop');
  background-size: cover !important;
  background-position: center !important;
}

.bg-snow .bg-container {
  background-color: #e6e9f0;
  background-image:
    linear-gradient(to bottom, rgba(255, 255, 255, 0.3), rgba(230, 233, 240, 0.6)),
    url('https://images.unsplash.com/photo-1491002052546-bf38f186af56?q=80&w=2000&auto=format&fit=crop');
  background-size: cover !important;
  background-position: center !important;
}

.bg-cloudy .bg-container {
  background-color: #8e9eab;
  background-image:
    linear-gradient(to bottom, rgba(0, 0, 0, 0.3), rgba(142, 158, 171, 0.6)),
    url('https://images.unsplash.com/photo-1534088568595-a066f410bcda?q=80&w=2000&auto=format&fit=crop');
  background-size: cover !important;
  background-position: center !important;
}

.bg-sunny .bg-container {
  background-color: #f8b500;
  background-image:
    linear-gradient(to bottom, rgba(248, 181, 0, 0.2), rgba(248, 181, 0, 0.4)),
    url('https://images.unsplash.com/photo-1522075469751-3a6694fb2f61?q=80&w=2000&auto=format&fit=crop');
  background-size: cover !important;
  background-position: center !important;
}

.bg-fog .bg-container {
  background-color: #b8d3d3;
  background-image:
    linear-gradient(to bottom, rgba(184, 211, 211, 0.4), rgba(184, 211, 211, 0.7)),
    url('https://images.unsplash.com/photo-1487621167305-5d248087c724?q=80&w=2000&auto=format&fit=crop');
  background-size: cover !important;
  background-position: center !important;
}

/* Add a semi-transparent overlay for content legibility */
.v-main {
  background-color: rgba(0, 0, 0, 0.15);
  backdrop-filter: blur(2px);
  /* Subtle blur for better text readability */
}

/* Ensure text remains readable */
.text-shadow {
  text-shadow: 0 2px 5px rgba(0, 0, 0, 0.7) !important;
}
</style>