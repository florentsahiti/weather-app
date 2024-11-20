<template>
  <div class="flex flex-col flex-1 items-center">
    <!-- Banner -->
    <div
      v-if="route.query.preview"
      class="text-white p-4 bg-weather-secondary w-full text-center"
    >
      <p>
        You are currently previewing this city, click the "+" icon to start tracking this city.
      </p>
    </div>

    <!-- Weather Overview -->
    <div v-if="weatherData" class="flex flex-col items-center text-white py-12">
      <h1 class="text-4xl mb-2">{{ route.params.city }}</h1>
      <p class="text-sm mb-12">
        {{
          new Date(weatherData.current_time).toLocaleDateString("en-us", {
            weekday: "short",
            day: "2-digit",
            month: "long",
          })
        }}
        {{
          new Date(weatherData.current_time).toLocaleTimeString("en-us", {
            timeStyle: "short",
          })
        }}
      </p>
      <p class="text-8xl mb-8">
        {{ Math.round(weatherData.current.temp) }}&deg;
      </p>
      <p>
        Feels like {{ Math.round(weatherData.current.apparent_temp) }}&deg;
      </p>
      <p class="capitalize">
        {{ weatherData.current.description }}
      </p>
      <img
        class="w-[150px] h-auto"
        :src="weatherData.current.icon"
        alt="weather icon"
      />
    </div>

    <hr class="border-white border-opacity-10 border w-full" />

    <!-- Hourly Weather -->
    <div v-if="weatherData" class="max-w-screen-md w-full py-12">
      <div class="mx-8 text-white">
        <h2 class="mb-4">Hourly Weather</h2>
        <div class="flex gap-10 overflow-x-scroll">
          <div
            v-for="(hourData, index) in weatherData.hourly"
            :key="index"
            class="flex flex-col gap-4 items-center"
          >
            <p class="whitespace-nowrap text-md">
              {{ new Date(hourData.time).toLocaleTimeString("en-us", { hour: "numeric" }) }}
            </p>
            <img class="w-auto h-[50px] object-cover" :src="hourData.icon" alt="hourly icon" />
            <p class="text-xl">{{ Math.round(hourData.temp) }}&deg;</p>
          </div>
        </div>
      </div>
    </div>

    <hr class="border-white border-opacity-10 border w-full" />

    <!-- Weekly Weather -->
    <div v-if="weatherData" class="max-w-screen-md w-full py-12">
      <div class="mx-8 text-white">
        <h2 class="mb-4">7 Day Forecast</h2>
        <div v-for="(day, index) in weatherData.daily" :key="index" class="flex items-center">
          <p class="flex-1">
            {{ new Date(day.date).toLocaleDateString("en-us", { weekday: "long" }) }}
          </p>
          <img class="w-[50px] h-[50px] object-cover" :src="day.icon" alt="daily icon" />
          <div class="flex gap-2 flex-1 justify-end">
            <p>H: {{ Math.round(day.max_temp) }}</p>
            <p>L: {{ Math.round(day.min_temp) }}</p>
          </div>
        </div>
      </div>
    </div>

    <!-- Remove City -->
    <div
      class="flex items-center gap-2 py-12 text-white cursor-pointer duration-150 hover:text-red-500"
      @click="removeCity"
    >
      <i class="fa-solid fa-trash"></i>
      <p>Remove City</p>
    </div>

    <!-- Loading State -->
    <div v-if="loading" class="text-white py-12">
      <p>Loading weather data...</p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";
import { useRoute, useRouter } from "vue-router";

const route = useRoute();
const router = useRouter();
const weatherData = ref(null);
const loading = ref(true);

const getWeatherData = async () => {
  const lat = route.query.lat;
  const lon = route.query.lng;

  if (!lat || !lon) {
    console.error("Latitude or Longitude is missing.");
    loading.value = false;
    return;
  }

  try {
    const response = await axios.get(
      `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&hourly=temperature_2m,weathercode&daily=temperature_2m_max,temperature_2m_min,weathercode&current_weather=true`
    );

    const data = response.data;
    weatherData.value = transformWeatherData(data);
    loading.value = false;
  } catch (err) {
    console.error("Error fetching weather data:", err);
    weatherData.value = null;
    loading.value = false;
  }
};

// Helper function to transform weather data
function transformWeatherData(data) {
  return {
    current: {
      temp: data.current_weather.temperature,
      apparent_temp: data.current_weather.apparent_temperature,
      description: "Current Conditions",
      icon: getWeatherIcon(data.current_weather.weathercode),
    },
    hourly: data.hourly.temperature_2m.map((temp, index) => ({
      time: data.hourly.time[index],
      temp: temp,
      icon: getWeatherIcon(data.hourly.weathercode[index]),
    })),
    daily: data.daily.temperature_2m_max.map((maxTemp, index) => ({
      date: data.daily.time[index],
      max_temp: maxTemp,
      min_temp: data.daily.temperature_2m_min[index],
      icon: getWeatherIcon(data.daily.weathercode[index]),
    })),
    current_time: data.current_weather.time,
  };
}

// Helper function for weather icons
function getWeatherIcon(weatherCode) {
  const iconMap = {
    0: "https://example.com/clear_day.png", // Clear sky
    1: "https://example.com/partly_cloudy.png", // Partly cloudy
    2: "https://example.com/cloudy.png", // Cloudy
    3: "https://example.com/overcast.png", // Overcast
    // Add more codes as needed
  };
  return iconMap[weatherCode] || "https://example.com/default.png";
}

// Get weather data when component is mounted
onMounted(() => {
  getWeatherData();
});

const removeCity = () => {
  const cities = JSON.parse(localStorage.getItem("savedCities")) || [];
  const updatedCities = cities.filter((city) => city.id !== route.query.id);
  localStorage.setItem("savedCities", JSON.stringify(updatedCities));
  router.push({ name: "home" });
};
</script>
