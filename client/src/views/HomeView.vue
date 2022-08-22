<template>
  <div class="h-screen relative">
    <GeoErrorModal @closeGeoError="closeGeoError" v-if="geoError" :geoErrorMsg="geoErrorMsg" />
    <MapFeatures @plotResult="plotResult" @getGeoLocation="getGeoLocation" 
    :coords="coords" :fetchCoords="fetchCoords" 
    @toggleSearchResults="toggleSearchResults" 
    :searchResults="searchResults" 
    @removeResult="removeResult"  />
    <div id="map" class="h-full z-[1]"></div>

  </div>
</template>

<script>
import leaflet from "leaflet"
import {onMounted, ref} from "vue"
import GeoErrorModal from '../components/GeoErrorModal.vue'
import MapFeatures from '../components/MapFeatures.vue'

export default {
  name: 'HomeView',
  components: { GeoErrorModal, MapFeatures },
  setup(){
    let map;
    onMounted(() =>{
      //init map
      map = leaflet.map('map').setView([51.505, -0.09], 13);

      //add title layer
    leaflet.tileLayer(`https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=${process.env.VUE_APP_API_KEY}`, {
      attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
      maxZoom: 18,
      id: 'mapbox/streets-v11',
      tileSize: 512,
      zoomOffset: -1,
      accessToken: process.env.VUE_APP_API_KEY,
    }).addTo(map);

    map.on("moveend", ()=>{
      closeSearchResults();
    })

    getGeoLocation();
    })
  
    const coords = ref(null);
    const fetchCoords = ref(null);
    const geoMarker = ref(null);
    const geoError = ref(null)
    const geoErrorMsg = ref(null)

    const getGeoLocation = () =>{
      if(coords.value){
        coords.value = null;
        sessionStorage.removeItem("coords");
        map.removeLayer(geoMarker.value);
        return;
      }
      //check session storage for coords
      if(sessionStorage.getItem('coords')){
        coords.value = JSON.parse(sessionStorage.getItem("coords"))
        plotGeoLocation(coords.value);
        return;
      }
      fetchCoords.value = true;
      navigator.geolocation.getCurrentPosition(setCoords, getLocError);
    };

    const setCoords = (pos)=>{
      //stop fetching coords
      fetchCoords.value = null;

      //set coords in session storage
      const setSessionCoords = {
        lat: pos.coords.latitude,
        lng: pos.coords.longitude,
      };

      sessionStorage.setItem('coords', JSON.stringify(setSessionCoords));

      //set ref coords value
      coords.value = setSessionCoords;

      plotGeoLocation(coords.value);
    };
    const getLocError = (err)=>{
      fetchCoords.value = null;
      geoError.value = true;
      geoErrorMsg.value = err.message;
    };
    const plotGeoLocation = (coords)=>{
      //create custom marker
      const customMarker = leaflet.icon({
        iconUrl: require('../assets/map-marker-red.svg'),
        iconSize: [35, 35]
      })
      //create new marker with coords and custom icon
      geoMarker.value = leaflet.marker([coords.lat, coords.lng], {icon: customMarker}).addTo(map);

      //set map view to current location
      map.setView([coords.lat, coords.lng], 10)
    };

    const closeGeoError = ()=>{
      geoError.value = null;
      geoErrorMsg.value = null;
    }
    const resultMarker = ref(null)
    const plotResult = (coords)=>{
      //check to see if result marker has value
      if(resultMarker.value){
        map.removeLayer(resultMarker.value)
      }
      //create custom marker
      const customMarker = leaflet.icon({
        iconUrl: require('../assets/map-marker-blue.svg'),
        iconSize: [35, 35]
      })
      //create new marker with coords and custom icon
      resultMarker.value = leaflet.marker([coords.coordinates[1], coords.coordinates[0]], {icon: customMarker}).addTo(map);

      //set map view to current location
      map.setView([coords.coordinates[1], coords.coordinates[0]], 13)
      closeSearchResults();
    }
    const removeResult = ()=>{
      map.removeLayer(resultMarker.value)
    }
    //close search result when interact to map
    const searchResults = ref(null);
    const toggleSearchResults = ()=>{
      searchResults.value = !searchResults.value;
    };
    const closeSearchResults = ()=>{
      searchResults.value = null
    }

    return {coords, fetchCoords, 
    getGeoLocation, geoMarker, 
    closeGeoError, geoError, geoErrorMsg, 
    plotResult, searchResults, toggleSearchResults, 
    closeSearchResults, removeResult};
  },
}
</script>
