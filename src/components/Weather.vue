<template>
  <div class="weather-app">
    <h1 class="app-title">Погодний додаток</h1>
    <div class="input-container">
      <div class="city-input-group">
        <h2>Додати місто</h2>
        <input class="city-input" v-model="cityInput" placeholder="Введіть місто" @input="getWeather" />
      </div>
      <div class="selected-city-group">
        <h2>Обрані міста</h2>
        <select class="selected-city-select" v-model="selectedCity" @change="getSelectedCityWeather">
          <option value="">Оберіть місто</option>
          <option v-for="city in selectedCities" :key="city" :value="city">{{ city }}</option>
        </select>
      </div>
    </div>
    <div class="button-container">
      <button class="add-city-button" @click="addCity">Обрати місто</button>
    </div>
    <div class="weather-details">
      <h2>Погода в місті {{ selectedCity }}</h2>
      <div class="weather-details-content">
        <template v-if="selectedCityWeather">
          <div class="weather-info">
            <p>Температура: {{ selectedCityWeather.main.temp }}°C</p>
            <p>Вологість: {{ selectedCityWeather.main.humidity }}%</p>
            <p>Тиск: {{ selectedCityWeather.main.pressure }}гПа</p>
            <p>Опис погоди: {{ selectedCityWeather.weather[0].description }}</p>
            <p>Швидкість вітру: {{ selectedCityWeather.wind.speed }} м/с</p>
          </div>
        </template>
        <template v-else>
          <p>Завантаження даних про погоду...</p>
        </template>
      </div>
    </div>
  </div>
</template>

<script setup>
import axios from "axios";
import { ref, reactive, onMounted } from "vue";

// Оголошення змінних і функцій
const cityInput = ref("");
const selectedCity = ref("");
const selectedCities = reactive([]);
const selectedCityWeather = ref(null);

// Отримання погоди за локацією користувача
const getWeatherByLocation = (position) => {
  const { latitude, longitude } = position.coords;
  axios.get(`https://api.openweathermap.org/data/2.5/weather?lat=${latitude}&lon=${longitude}&appid=7914d5a440960cfd5df3bd0388a7ad0f&units=metric`)
    .then((response) => {
      selectedCity.value = response.data.name;
      getSelectedCityWeather();
    })
    .catch((error) => {
      console.error("Помилка при отриманні даних про погоду за місцезнаходженням", error);
    });
};

// Отримання локації користувача
const getUserLocation = () => {
  if ("geolocation" in navigator) {
    navigator.geolocation.getCurrentPosition(getWeatherByLocation);
  } else {
    console.error("Геолокація недоступна в цьому браузері.");
  }
};

// Отримання погоди для вибраного міста
const getWeather = async () => {
  if (selectedCity.value) {
    try {
      const response = await axios.get(`https://api.openweathermap.org/data/2.5/weather?q=${selectedCity.value}&appid=7914d5a440960cfd5df3bd0388a7ad0f&units=metric`);
      selectedCityWeather.value = response.data;
    } catch (error) {
      console.error("Помилка при отриманні даних про погоду", error);
    }
  }
};

// Додавання міста до обраних
const addCity = () => {
  if (cityInput.value && !selectedCities.includes(cityInput.value)) {
    selectedCities.push(cityInput.value);
  }
  cityInput.value = "";
};

// Отримання погоди для вибраного міста
const getSelectedCityWeather = () => {
  if (selectedCity.value) {
    axios.get(`https://api.openweathermap.org/data/2.5/weather?q=${selectedCity.value}&appid=7914d5a440960cfd5df3bd0388a7ad0f&units=metric`)
      .then((response) => {
        selectedCityWeather.value = response.data;
      })
      .catch((error) => {
        console.error("Помилка при отриманні даних про погоду для обраного міста", error);
        selectedCityWeather.value = null;
      });
  }
};

// Отримання локації користувача при завантаженні сторінки
onMounted(() => {
 getUserLocation(); 
});
</script>

<style scoped>
/* Стилізація компонентів */
.weather-app {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  background: linear-gradient(180deg, #1c1c1c, #50007a, #1c1c1c);
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.2);
  color: #fff;
  border-radius: 10px;
}

.app-title {
  font-size: 24px;
  margin-bottom: 20px;
  text-align: center;
}

.input-container {
  display: flex;
  justify-content: space-between;
  margin-bottom: 20px;
  border-radius: 10px;
  padding: 20px;
}

.city-input-group, .selected-city-group {
  flex: 1;
  margin: 0 10px;
}

.city-input, .add-city-button, .selected-city-select {
  width: 100%;
  padding: 10px;
  font-size: 16px;
  background-color: transparent;
  color: #fff;
  border: none;
}

.add-city-button {
  background-color: #50007a;
  color: #fff;
  padding: 12px;
  font-size: 16px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: background-color 0.3s;
  margin: 10px auto 20px; 
}

.add-city-button:hover {
  background-color: #350055;
}

.selected-city-select {
  width: 100%;
  padding: 10px;
  font-size: 16px;
  background-color: transparent;
  color: #fff;
  border: none;
}

.selected-city-select:focus {
  border: 1px solid #007bff;
}

.selected-city-select option {
  background-color: rgba(0, 0, 0, 0.1); 
  color: #fff;
  transition: background-color 0.3s;
}

.selected-city-select option:hover {
  background-color: #007bff;
  color: #fff;
}

.weather-details {
  text-align: left;
  background: linear-gradient(180deg, rgba(44, 44, 44, 0.7), rgba(0, 0, 0, 0.7));
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.2);
}

.weather-info p {
  font-size: 16px;
  margin: 10px 0;
}

.weather-details-content p {
  font-size: 16px;
}

.button-container {
  text-align: center;
  margin-top: 20px;
}
</style>
