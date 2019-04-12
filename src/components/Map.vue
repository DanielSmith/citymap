<template>
  <div id="map"></div>
</template>

<script>
import $Scriptjs from 'scriptjs';
import { DEFAULT_ENCODING } from 'crypto';
export default {
  name: "Map",

  data() {
    return {
      map: null,
      marker: null,
      lastLat: null,
      lastLng: null,

      // center around the Upper West Side, NYC
      DEFAULT_LAT: 40.780,
      DEFAULT_LNG: -73.974
    }
  },

  mounted() {
    $Scriptjs("https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY&libraries=geometry,places", () => {
      this.initMap();
    });
  },

  methods: {
    initMap() {
      this.map = new google.maps.Map(document.getElementById("map"), {
        center: {
          lat: this.DEFAULT_LAT,
          lng: this.DEFAULT_LNG
        },
        zoom: 15,
        gestureHandling: "greedy"
      });

      google.maps.event.addListener(this.map, "click", (event) => {
        this.lastLat = event.latLng.lat();
        this.lastLng = event.latLng.lng();

        this.updateMarker(event.latLng);
      });
    },

    updateMarker(latLng) {
      if (!this.marker) {
        this.marker = new google.maps.Marker({
          map: this.map
        });
      }

      this.marker.setPosition(latLng);
    }
  }
}
</script>

<style>
#map {
  width: 90vw;
  height: 90vw;
}
</style>






