<template>
  <WeatherCard v-for="city in cities" :key="city" :city="city" />
  <Settings
    v-show="showSettings"
    :cities="cities"
    @updateCities="handleSettingsUpdateCities"
  />
  <img
    class="settings-switcher icon-as-button"
    @click="handleSettingsSwitcherClick"
    :src="showSettings ? CloseIcon : GearIcon"
    alt="Settings Icon"
  />
</template>

<script setup lang="ts">
import { ref } from "vue";
import Settings from "@/components/Settings.vue";
import GearIcon from "@/assets/gear.svg";
import CloseIcon from "@/assets/close.svg";
import WeatherCard from "@/components/WeatherCard.vue";
import { getCityData } from "@/api";

const showSettings = ref(false);

const cities = ref<string[]>(
  JSON.parse(localStorage.getItem("cities") || "[]")
);

const handleSettingsSwitcherClick = () => {
  showSettings.value = !showSettings.value;
};

const handleSettingsUpdateCities = (newCities: string[]) => {
  cities.value = newCities;
  localStorage.setItem("cities", JSON.stringify(newCities));
};

if (!cities.value.length) {
  navigator.geolocation.getCurrentPosition(
    async (pos) => {
      const { name, country } = await getCityData(
        pos.coords.latitude,
        pos.coords.longitude
      );
      cities.value.push(`${name}, ${country}`);
      localStorage.setItem("cities", JSON.stringify(cities.value));
    },
    async () => {
      const { name, country } = await getCityData("London");
      cities.value.push(`${name}, ${country}`);
      localStorage.setItem("cities", JSON.stringify(cities.value));
    }
  );
}
</script>

<style lang="scss">
@import "@/assets/styles.css";

weather-widget {
  position: relative;
  display: flex;
  flex-direction: column;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  width: 200px;
  row-gap: 20px;

  .settings-switcher {
    position: absolute;
    top: 4px;
    right: 0;
    width: 16px;
  }
}
</style>
