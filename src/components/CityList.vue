<script setup>
import axios from "axios";
import { ref, onMounted } from "vue";
import { useRouter } from "vue-router";
import CityCard from "./CityCard.vue";

const savedCities = ref([]);

// Helper function to map weather codes to icons and descriptions
const getWeatherIcon = (weatherCode) => {
  const iconMap = {
    0: "https://example.com/clear_day.png", // Clear sky
    1: "https://example.com/partly_cloudy.png", // Partly cloudy
    2: "https://example.com/cloudy.png", // Cloudy
    3: "https://example.com/overcast.png", // Overcast
    // Add more codes and icon mappings as needed
  };
  return iconMap[weatherCode] || "https://example.com/default.png"; // Default icon
};

// Function to get cities and weather data
const getCities = async () => {
  if (localStorage.getItem("savedCities")) {
    savedCities.value = JSON.parse(localStorage.getItem("savedCities"));

    const requests = savedCities.value.map((city) =>
      axios.get(
        `https://api.open-meteo.com/v1/forecast?latitude=${city.coords.lat}&longitude=${city.coords.lng}&hourly=temperature_2m&current_weather=true&timezone=auto`
      )
    );

    try {
      const weatherData = await Promise.all(requests);

      // Update each city object with the weather data received
      weatherData.forEach((value, index) => {
        savedCities.value[index].weather = {
          temp: value.data.current_weather.temperature,
          icon: getWeatherIcon(value.data.current_weather.weathercode),
          description: value.data.current_weather.weathercode, // You can map this to a description if needed
        };
      });
    } catch (err) {
      console.error("Error fetching weather data", err);
    }
  }
};

// Fetch the cities and weather data when the component is mounted
onMounted(() => {
  getCities();
});

const router = useRouter();

// Navigate to city view
const goToCityView = (city) => {
  router.push({
    name: "cityView",
    params: { state: city.state, city: city.city },
    query: {
      id: city.id,
      lat: city.coords.lat,
      lng: city.coords.lng,
    },
  });
};




</script>

