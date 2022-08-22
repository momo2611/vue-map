<template>
  <div class="w-full md:w-auto absolute md:top-[40px] md:left-[60px] z-[2] flex gap-4 px-6 py-8 md:px-0 md:py-0 bg-transparent">
      <!-- search -->
      <div class="relative flex-1 md:min-w-[350px]">
          <!-- input -->
          <input type="text" class="pl-9 pr-4 py-3 text-sm focus:outline-none w-full shadow-md rounded-md" 
          placeholder="Enter your address" v-model="searchQuery" 
          @input="search" @focus="$emit('toggleSearchResults')">
          <!-- search icon -->
          <div class="absolute top-0 left-[8px] h-full flex items-center">
              <i class="fas fa-search"></i>
          </div>
          <!-- search results -->
          <div class="absolute mt-2 w-full">
              <!-- results -->
              <div v-if="searchQuery && searchResults" class="h-[200px] overflow-scroll bg-white rounded-md">
                  <!-- loading -->
                  <LoadingSpinner v-if="!searchData" />
                  <div v-else>
                    <div v-for="(result, index) in searchData" :key="index" 
                    @click="selectResult(result)"
                    class="px-4 py-2 flex gap-x-2 cursor-pointer hover:bg-slate-600 hover:text-white">
                        <i class="fas fa-map-marker-alt"></i>
                        <p class="text-xs">{{result.place_name_en}}</p>
                    </div>
                  </div>
              </div>
              <!-- selected search result -->
              <div v-if="selectedResult" class="mt-[8px] px-4 py-3 bg-white rounded-md">
                  <i @click="removeResult" class="far fa-times-circle fex justify-end"></i>
                  <h1 class="text-lg">{{selectedResult.text}}</h1>
                  <p class="text-xs mb-1">{{selectedResult.properties.address}}, {{selectedResult.city}}, {{selectedResult.state}}</p>
                  <p class="text-xs">{{selectedResult.properties.category}}</p>
              </div>
          </div>
      </div>
    <!-- Geolocation -->
    <div class="px-4 bg-white flex items-center shadow-md rounded-md min-h-[45px]" :class="{'bg-slate-600': coords}">
        <i class="fas fa-location-arrow text-slate-6 text-[18px]" 
        :class="{'text-white': coords, 'animate-pulse': fetchCoords}" 
        @click="$emit('getGeoLocation')"></i>
    </div>
  </div>
</template>

<script>
import {ref} from 'vue'
import axios from 'axios'
import LoadingSpinner from './LoadingSpinner.vue'
export default {
    props: ['coords', "fetchCoords", "searchResults"],
    components: {LoadingSpinner},
    setup(props, {emit}){
        const searchQuery = ref(null)
        const searchData = ref(null)
        const queryTimeout = ref(null)
        const selectedResult = ref(null)
        // user typing
        const search = ()=>{
            clearTimeout(queryTimeout.value);

            searchData.value = null;
            queryTimeout.value = setTimeout(async ()=>{
                if(searchQuery.value !==""){
                    const params = new URLSearchParams({
                        fuzzyMatch: true,
                        language: "en",
                        limit: 10,
                        proximity: props.coords ? `${props.coords.lng},${props.coords.lat}` : "0,0",
                    });
                    const getData = await axios.get(`api/search/${searchQuery.value}?${params}`)
                    searchData.value = getData.data.features;
                }
            },750)
        }   

        const selectResult = (result) =>{
            selectedResult.value = result;
            emit('plotResult', result.geometry);
        };
        const removeResult = () =>{
            selectedResult.value = null;
            emit('removeResult');
        }

        return {searchQuery, searchData, 
        queryTimeout, search, 
        selectResult, selectedResult, removeResult}
    },
}
</script>