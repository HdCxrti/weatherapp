<template>
  <v-app id="inspire">
    <v-app-bar flat>
      <v-container class="mx-auto d-flex align-center justify-center">
        <v-avatar class="me-4 " color="grey-darken-1" size="32"></v-avatar>

        <v-btn v-for="link in links" :key="link" :text="link" variant="text"></v-btn>

        <v-spacer></v-spacer>

        <v-responsive max-width="260">
          <div class="d-flex align-center">
            <v-text-field v-model="searchCity" density="compact" label="Search city" rounded="lg" variant="solo-filled"
              flat hide-details single-line class="me-2" @keyup.enter="searchWeather"></v-text-field>
            <v-btn color="primary" icon @click="searchWeather" :loading="isSearching">
              <v-icon>mdi-magnify</v-icon>
            </v-btn>
          </div>
        </v-responsive>
      </v-container>
    </v-app-bar>

    <v-main class="bg-grey-lighten-3">
      <v-container>
        <router-view :city="currentCity" :key="searchKey" />
      </v-container>
    </v-main>
  </v-app>
</template>

<script setup>
import { ref, provide } from 'vue'

const links = [
  'Dashboard',
  'Messages',
  'Profile',
  'Updates',
]

const searchCity = ref('')
const currentCity = ref('London') // Default city
const isSearching = ref(false)
const searchKey = ref(0)

// Make city available to child components
provide('city', currentCity)

const searchWeather = () => {
  if (!searchCity.value.trim()) return

  isSearching.value = true
  currentCity.value = searchCity.value.trim()

  // Changing the key will force the child component to re-render
  searchKey.value++

  setTimeout(() => {
    isSearching.value = false
  }, 1000)
}
</script>