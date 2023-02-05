<template>
  <div>
    <div class="location-box">
      <Button @click="sendLocation()">
        <img src="/icons/location.png" alt="get-location" />
      </Button>
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      position: {
        lat: "",
        lon: "",
      },
    };
  },

  methods: {
    getGeoLocation() {
      navigator.geolocation.getCurrentPosition(
        (position) => {
          this.position.lat = position.coords.latitude;
          this.position.lon = position.coords.longitude;
        },

        (error) => {
          alert("Allow us to access this device location");
          console.log(error.message);
        }
      );
    },

    sendLocation() {
      this.$emit("userLocation", this.position);
    },
  },

  mounted() {
    this.getGeoLocation();
  },
};
</script>
<style scoped>
.location-box button {
  width: 6rem;
  height: 6rem;
  background-color: rgba(255, 255, 255, 0.25);
  border-radius: 50%;
  margin-left: 2rem;
  color: #fff;
  font-size: 1.5rem;
  font-weight: 400;
  outline: none;
  border: none;
  cursor: pointer;
  box-shadow: 1px 3px rgb(0 0 0 / 25%);
}
.location-box img {
  width: 4rem;
  height: 4rem;
}
.location-box button:focus {
  background-color: rgba(255, 255, 255, 0.75);
}

.location-box {
  transition: 0.6s;
  animation: bounce 2s infinite;
}

@keyframes bounce {
  0% {
    transform: translateY(2px);
  }
  50% {
    transform: translateY(-2px);
  }
  100% {
    transform: translateY(2px);
  }
}
</style>
