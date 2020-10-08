<template>
  <div class="container mx-auto py-5 px-5" >

    <div class="flex py-5 px-5 flex-wrap">
        <div class="inline-flex mr-auto py-4">
          <button @click="toggleType(1)" :class="listingType===1? 'toggle-active': 'toggle-inactive'" class="btn-toggle focus:outline-none">Buy</button>
          <button @click="toggleType(2)" :class="listingType===2? 'toggle-active': 'toggle-inactive'" class="btn-toggle focus:outline-none">Rent</button>
        </div>

        <div v-if="!isSearchTextValid" class="error-box">
          <strong class="align-middle"> Invalid input!</strong>
          <span class="align-middle"> Looks like you entered invalid character.</span>
        </div>

        <div class="inline-block ml-auto my-3">
          <input @input="debounceInput" v-model="searchText" class="search-bar focus:outline-none placeholder-gray-500" placeholder="Enter keyword.." type="text" >
        </div>
    </div>

    <div v-if=" houses.length > 0 ">
      <div class="flex flex-wrap">
        <house-cell v-for="item in houses" :key="item.id" :house="item"></house-cell>
      </div>

      <custom-pagination @pageClicked="setPage" class="mx-auto" :pagination="pagination"> </custom-pagination>
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
import customPagination from '~/components/custom-pagination.vue'
import {debounce} from 'lodash'

export default {
  components:{
    "house-cell": houseCell,
    "custom-pagination":customPagination
  },
  data(){
    return{
      houses: [],
      searchText:this.$route.query.searchText || "",
      limitPerPage:27,
      pagination:{
        page:this.$route.query.page || 1
      },
      listingType: 1
    }
  },
  computed:{
    isSearchTextValid(){
      const regex = /^[a-zA-Z0-9,. ]*$/;
      return (regex.test(this.searchText));
    }
  },
  methods:{
    toggleType(type){
      if(this.listingType !== type){
        this.listingType = type;
        this.pushQuery()
        if(this.isSearchTextValid){
          this.fetchHomes();
        }
      }
    },

    debounceInput: debounce(function(){
      if(this.isSearchTextValid){
        this.pushQuery()
        this.fetchHomes();
      }
    },400),

    setPage(clickedPage){
      this.pagination.page = clickedPage
      this.pushQuery()
      this.fetchHomes()
    },

    pushQuery(){
      let query;
      if(this.searchText){
        {
          query = {
            searchText: this.searchText,
            page:this.pagination.page,
            listingType: this.listingType
          }
        }
      }else{
        query = {
          page:this.pagination.page,
          listingType: this.listingType
        }
      }
      this.$router.push({query})
    },

    fetchHomes(){
      this.$axios
        .get("https://homehapp-api.jsteam.gaussx.com/api/home", {
          params:{
            searchText: this.searchText,
            listingTypes: this.listingType,
            page:this.pagination.page,
          }
        })
        .then(response=> {
          this.pagination=response.data.data.pagination

          if(this.listingType===1){
            this.houses = response.data.data.data.filter(house => {return(house.price !== null && house.city !== null)})
          }else{
            this.houses = response.data.data.data.filter(house => {return(house.rentPrice !== null && house.city !== null)})
          }
        })
        .catch(error => console.log(error))
    }

  },

  mounted() {
    this.setPage(this.pagination.page)
  }
}
</script>

<style>
  .btn-toggle{
   @apply text-xs font-bold font-sans uppercase w-24 border-b-4;
  }

  .toggle-active{
    @apply border-indigo-600 text-indigo-600;
  }

  .toggle-inactive{
    @apply border-gray-400 text-gray-500;
  }

  .error-box{
    @apply tracking-widest border border-red-400 text-red-700 rounded mx-auto text-xs font-semibold p-3 my-3;
  }

  .search-bar{
    @apply rounded-lg border-2 border-indigo-500 p-3;
  }
</style>
