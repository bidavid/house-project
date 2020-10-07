<template>
  <div class="container mx-auto py-5 px-5" >

    <div class="flex py-5 px-5 flex-wrap">

      <div class="inline-flex mr-auto py-4">
        <button @click="toggleType(1)" :class="listingType===1? 'btn-toggle-active': 'btn-toggle-inactive'" class="focus:outline-none">Buy</button>
        <button @click="toggleType(2)" :class="listingType===2? 'btn-toggle-active': 'btn-toggle-inactive'" class="focus:outline-none">Rent</button>
      </div>

      <div v-if="!searchTextValid" class="error-box">
        <strong class="align-middle"> Holy smokes!</strong>
        <span class="align-middle"> Looks like you entered invalid character.</span>
      </div>

      <div class="inline-block ml-auto my-3">
        <input @input="debounceInput" class="search-bar focus:outline-none placeholder-gray-500" placeholder="Enter keyword.." type="text" v-model="searchText">
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
  .btn-toggle-active{
    @apply text-gray-600 text-xs font-bold font-sans uppercase w-24 border-b-4 border-indigo-600 text-indigo-600;
  }

  .btn-toggle-inactive{
    @apply text-gray-600 text-xs font-bold font-sans uppercase w-24 border-b-4 border-gray-400 text-gray-500;
  }

  .error-box{
    @apply border border-red-400 text-red-700 rounded mx-auto text-sm p-3 my-3;
  }

  .search-bar{
    @apply rounded-lg border-2 border-indigo-500 p-3;
  }
</style>
