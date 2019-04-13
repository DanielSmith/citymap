<template>
  <v-flex xs4>
    <v-card>
      <div id="map"></div>
    </v-card>
  </v-flex>
</template>

<script>
import $Scriptjs from 'scriptjs';
import { eventBus } from '@/event-bus.js';


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
      DEFAULT_LNG: -73.974,

      geocoder: null
    }
  },

  mounted() {
    $Scriptjs("https://maps.googleapis.com/maps/api/js?key=AIzaSyBq4sydOK1dxIFjwITRJNp2rBq9hTBid4I&libraries=geometry,places", () => {
      this.initMap();
    });
  },

  methods: {
    initMap() {
      this.geocoder = new google.maps.Geocoder();

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

        this.geocoder.geocode({'location': event.latLng}, (results, status) => {
          console.log(results);

          const firstAddress = results[0].formatted_address;
          console.log(firstAddress);


          eventBus.$emit('mapAddress', results);
        });

      });

      // now we can init other things that depend on Google being loaded
      // tell the event bus
      eventBus.$emit('googleInit', {});
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






