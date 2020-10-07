<template>
  <div class="container mx-auto py-5 px-5" >

    <div class="flex py-5 px-5 flex-wrap">
        <div class="inline-flex mr-auto py-4">
          <button @click="toggleType(1)" :class="listingType===1? 'btn-toggle-active': 'btn-toggle-inactive'" class="focus:outline-none">Buy</button>
          <button @click="toggleType(2)" :class="listingType===2? 'btn-toggle-active': 'btn-toggle-inactive'" class="focus:outline-none">Rent</button>
        </div>

        <div v-if="!searchTextValid" class="error-box">
          <strong class="align-middle"> Invalid input!</strong>
          <span class="align-middle"> Looks like you entered invalid character.</span>
        </div>

        <div class="inline-block ml-auto my-3">
          <input @input="debounceInput" v-model="searchText" class="search-bar focus:outline-none placeholder-gray-500" placeholder="Enter keyword.." type="text" >
        </div>
    </div>

    <div v-if=" houses.length > 0 " class="flex flex-wrap">
      <house-cell v-for="item in houses" :key="item.id" :house="item"></house-cell>
    </div>

    <div v-else class="bg-yellow-100 border-t-4 border-orange-500 rounded-b text-orange-900 px-4 py-3 shadow-md">
      <div class="flex">
        <div class="py-1">
          <svg class="fill-current h-6 w-6 text-orange-500 mr-4" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20"><path d="M2.93 17.07A10 10 0 1 1 17.07 2.93 10 10 0 0 1 2.93 17.07zm12.73-1.41A8 8 0 1 0 4.34 4.34a8 8 0 0 0 11.32 11.32zM9 11V9h2v6H9v-4zm0-6h2v2H9V5z"/></svg>
        </div>
        <div>
          <p class="font-bold">0 objects fetched.</p>
          <p class="text-sm">No object matches your keyword.</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import houseCell from '~/components/house-cell.vue'
import {debounce} from 'lodash'

export default {
  components:{
    "house-cell": houseCell
  },
  data(){
    return{
      houses: [],
      searchText:"",
      limitPerPage:27,
      listingType: 1
    }
  },
  computed:{
    searchTextValid(){
      const regex = /^[a-zA-Z0-9,. ]*$/;
      return (regex.test(this.searchText));
    }
  },
  methods:{
    toggleType(type){
      if(this.listingType !== type){
        this.listingType = type;
        if(this.searchTextValid){
          this.fetchHomes();
        }
      }
    },

    fetchHomes(){
      this.$axios
        .get("https://homehapp-api.jsteam.gaussx.com/api/home", {
          params:{
            searchText: this.searchText,
            listingTypes: this.listingType,
            limit: this.limitPerPage
          }
        })
        .then(response=> {
          if(this.listingType===1){
            this.houses = response.data.data.data.filter(house => {return(house.price !== null && house.city !== null)})
          }else{
            this.houses = response.data.data.data.filter(house => {return(house.rentPrice !== null && house.city !== null)})
          }
        })
        .catch(error => console.log(error))
    },

    debounceInput: debounce(function(){
      if(this.searchTextValid){
        this.fetchHomes();
      }
    },400)

  },

  mounted() {
    this.fetchHomes()
  }
}
</script>

<style>
  .btn-toggle-active{
    @apply text-gray-600 text-xs font-bold font-sans uppercase w-24 border-b-4 border-indigo-600 text-indigo-600;
  }

  .btn-toggle-inactive{
    @apply text-gray-600 text-xs font-bold font-sans uppercase w-24 border-b-4 border-gray-400 text-gray-500;
  }

  .error-box{
    @apply tracking-widest border border-red-400 text-red-700 rounded mx-auto text-xs font-semibold p-3 my-3;
  }

  .search-bar{
    @apply rounded-lg border-2 border-indigo-500 p-3;
  }
</style>
