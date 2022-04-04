# citymap

## Intro

Search places on a map, filtering by City Boundaries.

I recently wrote a 5 part series that shows how to do mapping, and searching which can be filtered to within City Boundaries. I use Vue.js, Google Maps and Places, GeoJSON, and polygon filtering.  Citymap is a stripped down blend of a bunch of small parts.  I dont claim for it to be a shining illustration of UI, component layout, or error checking.  It's an end to end example, done in limited time.

![image](https://user-images.githubusercontent.com/167197/56677116-cb600400-668d-11e9-898b-dc5ef8cc2d55.png)

## Features

* Vue.js + Vuetify UI and component setup
* example of the use of Google Maps and Places API
* example of using Eventbus for communication
* example of using GeoJSON to get City Boundaries
* example of filtering search results against polygons

## The Articles

You can get to the articles via this [box set page](https://medium.com/@javajoint/how-to-map-cities-with-vue-geojson-and-google-box-set-7d9afd486070)


## Prerequisites

* Google API account set up.   You will be generating an API key for four access to four APIs: Geocoding, Geolocation, Maps, and Places:

![image](https://cdn-images-1.medium.com/max/800/1*X34nreXWo-Hd-aWAOpolfA.png)

See more about this in [Location, Location, Location (Part 2)](https://medium.com/@javajoint/how-to-map-cities-with-vue-geojson-and-google-part-2-9b8bdf540b96)

* Vue cli

* The instructions in [Set up Vue.js + Google Maps (Part 1)](https://medium.com/@javajoint/how-to-map-cities-with-vue-geojson-and-google-part-1-617d3eb796e0)


## Next Steps (code quality)

To make this more robust, there are several things that could be done:

I wrote this quickly, along with the articles that go with it.  I have some insight as to how to do several things better:

* better layout of Vue.js components - dont depend on App.vue for so much
* needs better error checking
* there is no notion of cleaning up data structures when going between cities - this could chew up memory quickly
* the method of finding the correct element in GeoJSON is not quite there.  I should look further for a Polygon or Multipolygon in some cases
* configuration data (specifically API keys) should be stored in an external file
* could make better use of Vuetify - this example is a bit ugly

## Next Steps (features)

To go beyond this example, in terms of features, I can hint at a few directions:

* be able to search among several APIs (not just Google Places)
* be able to search categories, in addition to keywords
* be able to specify search radius
* or, draw a polygon to search within
* be able to combine results from several APIs
* get more information from each search result (drill down for details)

Those are just some of the quick ideas.  I can think of many more, and there is some cross pollination I could do with my ThereThenThat and MediaShiskebab projects.


### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
