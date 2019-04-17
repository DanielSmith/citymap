<template>
  <v-app>
    <v-toolbar app>
      <v-toolbar-title class="headline">
        <span>CityMap</span>
      </v-toolbar-title>
    </v-toolbar>

    <!--
      TODO:  move the location input to its own component
      "pac" is "Place AutoComplete"
    -->
    <v-content>
      <v-container fluid>
        <v-layout row>
            <v-flex xs5>
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
            <v-btn v-if="curPlace"
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
      cityState: '',

      curPlace: null,
      geocoder: null
    }
  },

  mounted: function() {

    // targets for emitted events
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
      this.curPlace = '';
    },

    getGeoJSON() {
      console.log('get city');
      console.log(this.cityState);
    },


    cityStateHelper(str) {
      let city = '';
      let state = '';

      if (!str) return '';

      const addressParts = str.split(',');
      city = addressParts[0];
      // just the first two characters...
      state = addressParts[1].trim().substring(0,2);
      return `${city},${state}`;
    },

    getCityState(address) {
      let cs = '';

      // this is not foolproof.. do a deep dive into the address components
      // and their type, if you are going to be more thorough.. I am just
      // working off of the formatted address
      // more info at https://developers.google.com/maps/documentation/geocoding/start?csw=1
      address.map(curAddress => {        
        if (curAddress.types[0] === "locality") {
          cs = this.cityStateHelper(curAddress.formatted_address);
        }
      });
      return cs;
    },

    updateAddressFromMap(payload) {
      // TODO: error checking...
      const firstAddress = payload[0].formatted_address;

      this.theLocation = firstAddress;
      // if we have something here, it makes the 'get city' button visible
      this.curPlace = this.theLocation;

      // parse the address to get city and state
      this.cityState = this.getCityState(payload);
    },

    initMapAutocomplete() {
      this.geocoder = new google.maps.Geocoder();

      const input = document.getElementById('pac-input');
      const autocomplete = new google.maps.places.Autocomplete(input);

      google.maps.event.addListener(autocomplete, 'place_changed', () => {
        this.curPlace = autocomplete.getPlace();
        this.theLocation = this.curPlace.formatted_address;

        if (typeof this.curPlace.formatted_address !== "undefined") {          
          // parse the address to get city and state
          this.cityState = this.cityStateHelper(this.curPlace.formatted_address);

          // we need to update the map
          eventBus.$emit('newTextAddress', this.curPlace);
        }
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

