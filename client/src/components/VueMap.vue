<template lang="html">
  <div id="glasgowMap">
      {{showLocations()}}
  </div>

</template>

<script>
import {eventBus} from '@/main.js';
import PlaqueService from '@/services/PlaqueService'
import L from 'leaflet';
import 'leaflet-routing-machine';
import 'lrm-graphhopper';
import 'leaflet.locatecontrol';

export default {
  name: 'glasgowMap',
  components: {
    L
  },
  props: ['locations'],
  data() {
    return {
      zoom: 12,
      center: [55.860497, -4.257916],
      url: 'https://maps.heigit.org/openmapsurfer/tiles/roads/webmercator/{z}/{x}/{y}.png',
      attribution: '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors | <a href="https://www.graphhopper.com/">GraphHopper API</a> | <a href="https://www.liedman.net/leaflet-routing-machine/">Leaflet Routing Machine</a> | <a href="https://github.com/domoritz/leaflet-locatecontrol">Leaflet Locatecontrol</a>',
      longitude: "",
      latitude: "",
      userAdded: null
    }
  },
  mounted() {
    this.glasgowMap = L.map('glasgowMap');
    this.glasgowMap.setView(this.center, this.zoom);
    this.glasgowMap.options.minZoom = 11;
    L.tileLayer(this.url, {attribution: this.attribution}).addTo(this.glasgowMap);

    // Listener for clicks on new location
    this.glasgowMap.addEventListener('dblclick', (e) => {
      let coords = [e.latlng.lat, e.latlng.lng]
      this.addLocation(coords)
    });
    //end of Listener
    // Listener for user new location coming back with extra info
    this.glasgowMap.addEventListener(
      eventBus.$on('location-newmarker', (userLocation) => {
        L.marker([userLocation.latitude, userLocation.longitude]).addTo(this.glasgowMap)
        .bindPopup(userLocation.title + "<br />" + userLocation.address + "</div>", {maxWidth: 200, minWidth: 200, offset: [-121, 138]})
      })
    );
    // end of listener

    // ROUTING MACHINE AND GEOLOCATION
    L.control.locate().addTo(this.glasgowMap);

    let control = L.Routing.control({
      router: new L.Routing.GraphHopper('73834236-5649-4fc6-995f-0587acdd1eb9', {
        urlParameters: {
          vehicle: 'foot'
        }
      })
    }).addTo(this.glasgowMap);

    //Your location as start point
    this.glasgowMap.locate()
    .on('locationfound', function(e) {

      let beginLocation = {
        lat: e.latitude,
        lng: e.longitude
      };

      control.spliceWaypoints(0, 1, beginLocation)
    });

    //Plan Favourites Tour
    eventBus.$on('tour-locations', (location) => {

      let counter = 1;

      for (let i = 0; i < location.length; i++) {
        control.spliceWaypoints(counter, 1, location[i])
        counter++
      }

      eventBus.$on('tour-deleted', () => {
        control.spliceWaypoints(1, location.length)
      })

    });
    // Route to a specific plaque
    eventBus.$on('route-end', (endLocation) => {

      let endLatLng = {
        lat: endLocation[0],
        lng: endLocation[1]
      }
      control.spliceWaypoints(control.getWaypoints().length - 1, 1, endLatLng)

      eventBus.$on('tour-deleted', () => {
        control.spliceWaypoints(1, 1)
      })
    });
  },
  methods: {
    showLocations(){
      for (let i = 0; i < this.locations.length; i++) {
        if (this.locations[i].latitude || this.locations[i].longitude !== null) {
          L.marker([this.locations[i].latitude, this.locations[i].longitude], {title: this.locations[i].title.capitalize, alt: this.locations[i].title, riseOnHover: true})
          .addTo(this.glasgowMap)
          .bindPopup("<div id=" + this.locations[i]._id + " class=" + this.locations[i].colour_name + "><b>" + this.locations[i].title.toUpperCase() + "</b><br />"
          + this.locations[i].address + "</div>", {maxWidth: 200, minWidth: 200, offset: [-121, 138]})
          .on("click", function(marker) {
            let location = marker.latlng;
            eventBus.$emit('location-selected', location)
            eventBus.$emit('option-selected', 'details')
            eventBus.$emit('show-toggle', false)
          });
        }
      }
    },
    addLocation(coords) {
      const payload = {
        latitude: coords[0],
        longitude: coords[1],
        userAdded: true
      };
      eventBus.$emit('location-added', payload);
      eventBus.$emit('option-selected', 'update');
    }
  }
}
</script>

<style lang="css" scoped>
@import "~leaflet/dist/leaflet.css";
@import "../assets/leaflet-routing-machine.css";

#glasgowMap {
  width: 100vw;
  height: 100vh;
}

</style>
