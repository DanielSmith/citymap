<template>
  <v-app>
    <v-toolbar app>
      <v-toolbar-title class="headline">
        <span>CityMap</span>
      </v-toolbar-title>
    </v-toolbar>

    <!-- TODO:  move the location input to its own component -->
    <v-content>
      <v-container fluid>
        <v-layout row>
            <v-flex xs9>
              <v-text-field
                solo
                label="Location"
                placeholder=" "
                append-icon="close"
                @click:append="doClearLocation"
                v-model="theLocation"
                id="pac-input"
              >
              </v-text-field>
            </v-flex>
            <v-btn
              color="green"
              @click="getGeoJSON"
            >
              Get City
            </v-btn>
        </v-layout>

        <Map/>

      </v-container>
    </v-content>
  </v-app>
</template>

<script>
import Map from './components/Map'
import { eventBus } from '@/event-bus.js';


export default {
  name: 'App',
  components: {
    Map
  },
  data () {
    return {
      theLocation: '',

      geocoder: null
    }
  },

  mounted: function() {
    eventBus.$on('googleInit', () => {
      this.initMapAutocomplete();
    });

    eventBus.$on('mapAddress', (payload) => {
      this.updateAddressFromMap(payload);
    });
  },

  methods: {

    doClearLocation() {
      this.theLocation = '';
    },

    getGeoJSON() {

    },

    updateAddressFromMap(payload) {
      const firstAddress = payload[0].formatted_address;


      this.theLocation = firstAddress;

      // parse the address to get city and state, so that
      // we have something to fetch our city boundaries with
    },

    initMapAutocomplete() {
      this.geocoder = new google.maps.Geocoder();
      console.log(this.geocoder);

      let input = document.getElementById('pac-input');
      const autocomplete = new google.maps.places.Autocomplete(input);

      google.maps.event.addListener(autocomplete, 'place_changed', () => {
        const place = autocomplete.getPlace();

        console.log(place);
      });
    }
  }
}
</script>

<style>

.v-input__icon--append {
  cursor: pointer;
  pointer-events: auto;
}

</style>

