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

  mounted: function() {
    // use your own key...
    $Scriptjs("https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY&libraries=geometry,places", () => {
      this.initMap();
    });

    eventBus.$on('newTextAddress', (payload) => {
      this.updateFromTextAddress(payload);
    });

    eventBus.$on('sendCityData', (payload) => {
      this.renderCityMap(payload);
    })
  },

  methods: {
    updateFromTextAddress(payload) {
      this.map.setCenter(payload.geometry.location);
      this.updateMarker(payload.geometry.location);
    },


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
          const firstAddress = results[0].formatted_address;
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
    },

    renderCityMap(myCityData) {
      myCityData.setMap(this.map);

      // and pass clicks on to the underlying map
      myCityData.addListener('click', (event) => {
        google.maps.event.trigger(this.map, 'click', event);
      });
    }
  }
}
</script>

<style>
#map {
  width: 90vw;
  height: 50vw;
}
</style>






 