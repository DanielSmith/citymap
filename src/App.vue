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

            <v-flex xs3>
              <v-text-field
                solo
                label="Keywords"
                placeholder=" "
                v-model="theKeyword"
                id="ps-input"
              >
              </v-text-field>
            </v-flex>
            
            <v-btn
              color="green"
              @click.stop="launchPlacesSearch"
            >
              Search
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
import axios from 'axios';

export default {
  name: 'App',
  components: {
    Map
  },

  data () {
    return {
      theLocation: '',
      theKeyword: '',
      cityState: '',

      curPlace: null,
      geocoder: null,
      myCityData: null
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

    // toss this over to the Map component
    launchPlacesSearch() {
      eventBus.$emit('launchKeywordSearch', this.theKeyword)
    },

    doClearLocation() {
      this.theLocation = '';
      this.curPlace = '';
    },

    getGeoJSON() {
      this.getCityData(this.cityState);
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

    getCityData(theCity) {

      let apiPath = "https://nominatim.openstreetmap.org/search.php";

      let params = {
        q: theCity,
        polygon_geojson: 1,
        format: "json"
      };

      axios.get(apiPath, { params: params }  )
        .then(response => {
          let geoJSONDataChunk = response.data[0];


          // geojson data from http://nominatim.openstreetmap.org/ needs
          // to be wrapped, so that the google addGeoJson() call
          // can handle it properly
          const geoConf = {
            "type": "FeatureCollection",
            "features": [
              { "type": "Feature",
                "geometry": geoJSONDataChunk.geojson,
                "id": "city"
              }
            ]
          };

          this.myCityData = new google.maps.Data();
          this.myCityData.addGeoJson(geoConf, "city");
          this.myCityData.setStyle({
            fillColor: 'green',
            fillOpacity: 0.1,
            strokeWeight: 1
          });

          // send data to our Map component
          eventBus.$emit('sendCityData', this.myCityData);
        })
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

