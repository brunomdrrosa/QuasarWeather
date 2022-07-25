<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md">
      <q-input
        v-model="search"
        @keyup.enter="getWeatherBySearch"
        placeholder="Pesquisar"
        dark
        borderless
      >
        <template v-slot:before>
          <q-icon @click="getLocation" name="place" />
        </template>

        <template v-slot:append>
          <q-btn @click="getWeatherBySearch" round dense flat icon="search" />
        </template>
      </q-input>
    </div>
    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">{{ weatherData.name }}</div>
        <div class="text-h6 text-weight-light">
          {{ weatherData.weather[0].description.charAt(0).toUpperCase() + weatherData.weather[0].description.substring(1, weatherData.weather[0].description.length) }}
        </div>
        <div class="text-h1 text-weight-thin q-my-lg">
          <span>{{ Math.round(weatherData.main.temp) }}</span>
          <span class="text-h4 relative-position degree">&deg;C</span>
        </div>
      </div>
      <div class="col text-center">
        <img
          :src="`http://openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`"
        /></div
    ></template>

    <template v-else>
      <div class="col column text-center text-white">
        <div class="col text-h2 text-weight-thin">Quasar<br />Weather</div>
        <q-btn @click="getLocation" class="col" flat>
          <q-icon left size="3em" name="place" />
          <div>Encontrar minha localização</div>
        </q-btn>
      </div>
    </template>

    <div class="col skyline"></div>
  </q-page>
</template>

<script lang="ts">

export default {
  name: "IndexPage",
  data() {
    return {
      search: "",
      weatherData: null,
      lat: null,
      lon: null,
      apiUrl: "https://api.openweathermap.org/data/2.5/weather",
      apiKey: process.env.API_KEY,
    };
  },

  computed: {
    bgClass() {
      if (this.weatherData) {
        if (this.weatherData.weather[0].icon.endsWith("n")) {
          return "bg-night";
        }
        return "bg-day";
      }
    },
  },

  methods: {
    getLocation() {
      this.$q.loading.show();

      if (this.$q.platform.is.electron) {
        this.$axios.get(`http://ip-api.com/json/`).then((response) => {
          this.lat = response.data.lat;
          this.lon = response.data.lon;
          this.getWeatherByCoords();
        });
      }

      navigator.geolocation.getCurrentPosition((position) => {
        this.lat = position.coords.latitude;
        this.lon = position.coords.longitude;
        this.getWeatherByCoords();
      });
    },

    getWeatherByCoords() {
      this.$q.loading.show();
      this.$axios(
        `${this.apiUrl}?lat=${this.lat}&lon=${this.lon}&appid=${this.apiKey}&units=metric&lang=pt_br`
      ).then((response) => {
        this.weatherData = response.data;
        this.$q.loading.hide();
      });
    },

    getWeatherBySearch() {
      this.$q.loading.show();
      this.$axios(
        `${this.apiUrl}?q=${this.search}&appid=${this.apiKey}&units=metric&lang=pt_br`
      ).then((response) => {
        this.weatherData = response.data;
        this.$q.loading.hide();
      });
    },
  },
};
</script>

<style lang="sass">
.q-page
  background: linear-gradient(to bottom, #136a8a, #267871)
  &.bg-night
    background: linear-gradient(to bottom, #232526, #414345)
  &.bg-day
    background: linear-gradient(to bottom, #00b4db, #0083b0)

.degree
  top: -44px

.skyline
  flex: 0 0 100px
  background: url(../assets/skyline.png)
  background-size: contain
  background-position: center bottom
</style>
