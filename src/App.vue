<template>
  <div id="app" :style="{ backgroundImage: 'url(' + image + ')' }">
    <main>
      <div :class="{ active: isActive }">
        <div class="error">Please Enter correct city name</div>
      </div>

      <div class="search-box">
        <geo-location v-on:userLocation="getUserLocation"></geo-location>
        <input
          type="text"
          class="search-bar"
          placeholder="Search..."
          v-model="query"
          @keyup.enter="fetchWeather"
        />
      </div>

      <div :class="{ active: isHidden }">
        <div class="weather-wrap">
          <div class="location-box">
            <div class="left-side">
              <div class="left-side-inner">
                <div class="location">
                  {{ weather.name }} {{ weather.country }}
                </div>
                <div class="date">{{ dateBuilder() }}</div>
                <div class="date">{{ time }}</div>
              </div>
              <div class="icon-box">
                <img
                  :src="`http://openweathermap.org/img/wn/${weather.icon}@4x.png`"
                  :id="weather.id"
                />
              </div>
            </div>

            <div class="weather">{{ weather.sky }}</div>
            <div class="weather-data">
              <div class="weather-data wind">
                Wind Speed:
                {{ Math.round(weather.speed * coversion) + "  " + value }}
              </div>
              <div class="weather-data humidity">
                Humidity: {{ weather.humidity + " " + "%" }}
              </div>
              <div class="weather-data pressure">
                Pressure: {{ weather.pressure + " " + "hpa" }}
              </div>
              <div class="weather-data visibility">
                Visibility: {{ weather.visibility / 1000 + " " + "KM" }}
              </div>
              <div class="weather-data description">
                Description: {{ weather.description }}
              </div>
            </div>
          </div>
          <div class="weather-box">
            <div class="converter-box">
              <input
                type="checkbox"
                id="checkbox"
                v-model="picked"
                @change="uniteChange"
              />
              <label for="checkbox">{{ picked }}</label>
              <div class="degree"></div>
            </div>
            <div class="temp">{{ Math.round(weather.temp) }}°</div>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<script>
import axios from "axios";
import geo from "./components/geoLocation.vue";
import geoLocation from "./components/geoLocation.vue";
export default {
  components: { geoLocation },
  name: "App",
  data() {
    return {
      query: "",
      api_key: "fce9959711d99ead556e76eee490f8dd",
      image: "/img/clouds.jpg",
      isActive: true,
      isHidden: true,
      picked: "true",
      unit: "metric",
      coversion: 3.6,
      value: "Km/h",
      time: "",
      weather: {
        name: "",
        id: "",
        icon: "",
        speed: "",
        pressure: "",
        humidity: "",
        visibility: "",
        description: "",
        temp: "",
        country: "",
        sky: "",
        timeZone: "",
      },
    };
  },
  Components: {
    geoLocation: geo,
  },
  methods: {
    async fetchWeather() {
      try {
        const weatherData = await axios.get(
          `https://api.openweathermap.org/data/2.5/weather?q=${this.query}&appid=${this.api_key}&units=${this.unit}`
        );

        (this.weather.name = weatherData.data.name),
          (this.weather.icon = weatherData.data.weather[0].icon),
          (this.weather.id = weatherData.data.weather[0].id),
          (this.weather.speed = weatherData.data.wind.speed),
          (this.weather.pressure = weatherData.data.main.pressure),
          (this.weather.humidity = weatherData.data.main.humidity),
          (this.weather.visibility = weatherData.data.visibility),
          (this.weather.description = weatherData.data.weather[0].description),
          (this.weather.temp = weatherData.data.main.temp),
          (this.weather.sky = weatherData.data.weather[0].main);
        this.weather.country = weatherData.data.sys.country;
        this.weather.timeZone = weatherData.data.timezone;

        this.getAreaTime();
        this.isHidden = false;
      } catch (err) {
        this.isActive = false;
        setTimeout(() => {
          this.isActive = true;
        }, 1500);
      }
    },

    dateBuilder() {
      let d = new Date();
      let months = [
        "january",
        "february",
        "March",
        "april",
        "May",
        "June",
        "July",
        "August",
        "September",
        "October",
        "November",
        "December",
      ];
      let days = [
        "Sunday",
        "Monday",
        "Tuesday",
        "Wednesday",
        "Thursday",
        "Friday",
        "Saturday",
      ];
      let day = days[d.getDay()];
      let date = d.getDate();
      let month = months[d.getMonth()];
      let year = d.getFullYear();

      return `${day} ,${date} , ${month} ${year}`;
    },

    uniteChange() {
      if (this.picked === false) {
        this.unit = "imperial";
        this.coversion = 1;
        this.value = "mph";
      } else {
        this.unit = "metric";
        this.value = "Km/h";
        this.coversion = 3.6;
      }
    },

    getUserLocation(e) {
      let { lat, lon } = e;
      this.callAnotherApi(lat, lon);
      if (!lat && !lon) {
        alert("Allow us to access this device location");
      } else {
        this.callAnotherApi(lat, lon);
      }
    },

    async callAnotherApi(lat, lon) {
      const newData = await axios.get(
        `https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=${this.api_key}`
      );
      this.query = newData.data.name;
      this.fetchWeather();
    },

    getAreaTime() {
      let time = this.weather.timeZone;
      let d = new Date();
      let localTime = d.getTime();
      let localOffset = d.getTimezoneOffset() * 60000;
      let utc = localTime + localOffset;
      var locationTime = utc + 1000 * time;
      let day = new Date(locationTime).getHours();
      let data = new Date(locationTime)
        .toLocaleTimeString()
        .replace(/([\d]+:[\d]{2})(:[\d]{2})(.*)/, "$1$3");
      this.time = data;
      this.changeImage(day);
    },

    changeImage(day) {
      let days =
        (day < 12 && "Morning") || (day < 18 && "Afternoon") || "Evening";

      if (days === "Morning") {
        this.image = "/img/morning.jpg";
      } else if (days === "Afternoon") {
        this.image = "/img/afternoon.jpg";
      } else if (days === "Evening") {
        this.image = "/img/evening.jpg";
      }
    },
  },

  watch: {
    unit() {
      this.fetchWeather();
    },
  },
};
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: inherit;
}
html {
  font-size: 62.5%;
}
body {
  font-family: "Montserrat", sans-serif;
  box-sizing: border-box;
}
#app {
  background-size: cover;
  background-position: center;
  transition: 0.5s;
}
main {
  min-height: 100vh;
  padding: 25px;
  background-image: linear-gradient(
    to bottom,
    rgba(0, 0, 0, 0.25),
    rgba(0, 0, 0, 0.75)
  );
}

.search-box {
  width: 100%;
  /* margin-bottom: 3rem; */
  text-align: center;
  display: flex;
  flex-direction: row-reverse;
  margin: 3rem auto;
}
.search-box .search-bar {
  display: block;
  width: 100%;
  padding: 1.5rem;
  font-size: 1.5rem;
  color: #313131;
  border: none;
  background: none;
  appearance: none;
  outline: none;
  background-color: rgba(255, 255, 255, 0.15);
  text-transform: capitalize;
  border-radius: 3.5rem;
  transition: 0.4s;
  box-shadow: 0px 0px 8px rgba(0, 0, 0, 0.25);
}
.search-box .search-bar:focus {
  background-color: rgba(255, 255, 255, 0.75);
  box-shadow: 0px 0px 16px rgba(0, 0, 0, 0.25);
  border-radius: 16px 0px 16px 0px;
}
.weather-wrap {
  width: 80vw;
  min-height: 80vh;
  background: none;
  background-color: rgba(255, 255, 255, 0.15);
  box-shadow: 0px 0px 8px rgba(0, 0, 0, 0.25);
  margin: 0 auto;
  display: flex;
  justify-content: space-between;
  padding: 2rem 4rem;
}
.location-box .location {
  color: #fff;
  font-size: 5rem;
  font-weight: 500;
  text-transform: capitalize;
}
.location-box .date {
  color: #fff;
  font-size: 1.8rem;
  font-weight: 500;
}
.icon-box img {
  animation: floating 4s infinite;
}

@keyframes floating {
  0% {
    transform: translateX(5px);
  }
  50% {
    transform: translateX(-5px);
  }
  100% {
    transform: translateX(5px);
  }
}

.weather-data {
  color: #fff;
  font-size: 1.2rem;
  font-weight: 500;
  margin-top: 1rem;
}

.weather {
  color: #fff;
  font-size: 3rem;
  font-weight: 600;
  margin: 3rem 0;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}
.weather-box {
  text-align: center;
}
.weather-box .temp {
  display: inline-block;
  font-size: 10rem;
  color: #fff;
  text-shadow: 3px 6px rgb(0, 0, 0, 0.25);
  border-radius: 1.6rem;
  font-weight: 100;
}

.converter-box {
  width: 100%;
  height: 2rem;
  padding: 2rem;
  display: flex;
  align-items: center;
  justify-content: flex-end;
}
input[type="checkbox"] {
  height: 0;
  width: 0;
  visibility: hidden;
}

label {
  content: "";
  cursor: pointer;
  text-indent: -9999px;
  width: 5rem;
  height: 2rem;
  background: 3px 6px rgba(255, 255, 255, 0.25);
  display: block;
  border-radius: 100px;
  position: relative;
}

label:after {
  content: "";
  position: absolute;
  top: 0px;
  left: 30px;
  width: 2rem;
  height: 2rem;
  background: #fff;
  border-radius: 90px;
  transition: 0.3s;
  z-index: 100;
}

.converter-box .degree:before {
  content: "F";
  position: relative;
  top: 0px;
  left: -1.8rem;
  z-index: 10;
  font-size: 1.5rem;
  color: #fff;
  font-weight: 400;
}
.converter-box .degree:after {
  content: "C";
  position: relative;
  top: 0px;
  left: -5rem;
  z-index: 10;
  font-size: 1.5rem;
  color: #fff;
  font-weight: 400;
}

input[type="checkbox"]:not(:checked) + label {
  background: #bada55;
}

input:checked + label:after {
  left: calc(100% - -10px);
  transform: translateX(-300%);
}
.error {
  position: fixed;
  top: 0;
  left: 50%;
  -webkit-transform: translateX(-50%);
  transform: translateX(-50%);
  z-index: 9999;
  color: #fff;
  font-size: 1.8rem;
  font-weight: 400;
  text-align: center;
  border-bottom-left-radius: 5px;
  border-bottom-right-radius: 5px;
  padding: 1.6rem 15rem;
  -webkit-box-shadow: 0 2rem 4rem rgba(0, 0, 0, 0.25);
  box-shadow: 0 2rem 4rem rgba(0, 0, 0, 0.25);
  background-color: #eb4d4b;
  transition: 0.4s;
}
.active {
  display: none;
}

@media only screen and (max-width: 768px) {
  html {
    font-size: 58.5%;
  }
  .weather-data {
    font-size: 1.6rem;
  }

  .converter-box {
    padding: 0;
    margin-left: 3rem;
  }
}

@media only screen and (max-width: 670px) {
  .left-side {
    display: flex;
    flex-direction: column-reverse;
  }
  .left-side-inner {
    display: flex;
    flex-direction: column;
  }
  .weather-wrap {
    flex-direction: column-reverse;
  }
  .icon-box {
    display: flex;
    justify-content: center;
  }
}

@media only screen and (max-width: 500px) {
  html {
    font-size: 49.5%;
  }
  label:after {
    left: 22px;
  }
  .weather-wrap {
    width: 88vw;
  }
}
@media only screen and (max-width: 400px) {
  .weather-wrap {
    width: 90vw;
    padding: 2rem 3rem;
  }
}
</style>
