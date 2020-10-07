<template>
  <div class="container mx-auto py-5">

    <div class="flex mb-20 mt-5 h-12">

      <div class="inline-flex h-8">
        <button @click="toggleType(1)" :class="listingType===1? 'border-purple-600': 'border-purple-200'" class="btn-toggle focus:outline-none">Buy</button>
        <button @click="toggleType(2)" :class="listingType===2? 'border-purple-600': 'border-purple-200'" class="btn-toggle focus:outline-none">Rent</button>
      </div>

      <div v-if="!searchTextValid" class="inline-block border border-red-400 text-red-700 px-4 py-3 rounded mx-auto">
        <strong class="font-bold">Holy smokes!</strong>
        <span class="block sm:inline">Looks like you entered invalid character. </span>
      </div>

      <div class="inline-block ml-auto">
        <input @input="debounceInput" class="rounded-lg border-2 border-purple-300 p-3 focus:outline-none placeholder-purple-300" placeholder="Enter keyword.." type="text" v-model="searchText">
      </div>
    </div>

    <div class="flex flex-wrap">
      <house-cell v-for="item in houses" :key="item.id" :house="item"></house-cell>
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
      limitPerPage:18,
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
        this.fetchHomes();
      }else return
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
            this.houses = response.data.data.data.filter(house => house.price !== null)
          }else{
            this.houses = response.data.data.data.filter(house => house.rentPrice !== null)
          }
        })
        .catch(error => console.log(error))
    },

    debounceInput: debounce(function(){
      if(this.searchTextValid){
        this.fetchHomes()
      }
    },400)

  },

  mounted() {
    this.fetchHomes()
  }
}
</script>

<style>
  .btn-toggle{
    @apply text-gray-600 text-xs font-bold font-sans uppercase w-24 border-b-4;
  }
</style>
