<template>
  <div class="mx-auto py-5 px-5" >

    <div class="flex py-5 px-5 flex-wrap">
        <div class="inline-flex md:mr-auto py-4">
          <button :disabled="!isSearchTextValid" @click="toggleType(1)" :class="filtering.listingType===1? 'toggle-active': 'toggle-inactive'" class="btn-toggle focus:outline-none">Buy</button>
          <button :disabled="!isSearchTextValid" @click="toggleType(2)" :class="filtering.listingType===2? 'toggle-active': 'toggle-inactive'" class="btn-toggle focus:outline-none">Rent</button>
        </div>

        <div v-if="!isSearchTextValid" class="error-box">
          <strong class="align-middle"> Invalid input!</strong>
          <span class="align-middle"> Looks like you entered invalid character or whitespace. Filtering, paging and toggling disabled.</span>
        </div>

        <div class="inline-block md:ml-auto my-3 md:w-1/4">
          <input @input="debounceInput" v-model="filtering.searchText" class="search-bar focus:outline-none placeholder-gray-500" placeholder="Enter keyword.." type="text" >
        </div>

        <button @click="toggleFiltering" :disabled="!isSearchTextValid" :class="isSearchTextValid? 'opacity-100 hover:border-orange-600 hover:bg-orange-600':'opacity-75 cursor-default'" class="btn-open-filters md:ml-5 focus:outline-none">
          <img class="icon-filter" src="/images/filter_icon.png">
        </button>

    </div>

    <div v-if="houses.length > 0 ">
      <div class="flex flex-wrap">
        <house-cell v-for="item in houses" :key="item.id" :house="item"></house-cell>
      </div>

      <custom-pagination v-if="isSearchTextValid" @pageNumberClicked="setPage" class="mx-auto" :pagination="pagination"> </custom-pagination>
    </div>

    <div v-else class="notification-box">
      <div class="flex">
        <div class="py-1">
          <svg class="fill-current h-6 w-6 text-indigo-500 mr-4" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20"><path d="M2.93 17.07A10 10 0 1 1 17.07 2.93 10 10 0 0 1 2.93 17.07zm12.73-1.41A8 8 0 1 0 4.34 4.34a8 8 0 0 0 11.32 11.32zM9 11V9h2v6H9v-4zm0-6h2v2H9V5z"/></svg>
        </div>
        <div>
          <p class="font-bold">0 objects fetched.</p>
          <p class="text-sm">No object matches your keyword.</p>
        </div>
      </div>
    </div>

    <div v-if="filteringActive" class="absolute top-0 left-0 w-full h-screen flex flex-wrap">
      <div @click="dismissFiltering" class="h-full w-2/3 opacity-50 m-0 bg-gray-500 ">
      </div>
      <div class="h-full w-1/3 bg-orange-500 flex place-content-center">
        <div class="h-auto w-3/5 mx-auto my-24">
          <div class="mb-3">
            <span class="text-white tracking-widest font-sans text-lg block">Min. bedrooms: </span>
            <input class="filter-bar focus:outline-none w-1/5" v-model.number="filtering.bedrooms" type="number" min="0">
          </div>
          <div class="mb-3">
            <span class="text-white tracking-widest font-sans text-lg block">Min. bathrooms: </span>
            <input class="filter-bar focus:outline-none w-1/5" v-model.number="filtering.bathrooms" type="number" min="0">
          </div>
          <button @click="setFilters" :disabled="!areFiltersValid" :class="areFiltersValid? 'opacity-100 hover:shadow-l hover:border-indigo-600 hover:bg-indigo-600':'opacity-75 cursor-default'" class="btn-save-filters focus:outline-none">
            <span class="text-white uppercase text-sm font-semibold font-sans tracking-widest">Save</span>
          </button>
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
    "custom-pagination":customPagination,
  },
  data(){
    return{
      filteringActive: false,
      houses: [],
      filtering:{
        searchText:this.$route.query.searchText || "",
        listingType: parseInt(this.$route.query.listingType) || 1,
        bathrooms:parseInt(this.$route.query.bathrooms)||0,
        bedrooms:parseInt(this.$route.query.bathrooms)||0
      },
      pagination:{
        page: parseInt(this.$route.query.page) || 1
      },

      //kada listing type postavim na this.route.query.params ili 1
      //ako refresham klasa na toggle se ne postavlja, niti 1 home se ne povlaci
      //PAGE SMIJE BITI STRING I DOBIT CU PUN ARRAY KUCA, ALI LISTING TYPE STRING CE VRATIT PRAZNO POLJE KUCA

    }
  },
  computed:{
    isSearchTextValid(){
      const regex = /^[a-zA-Z0-9,.]*$/;
      return (regex.test(this.filtering.searchText));
    },

    areFiltersValid(){
      return (this.filtering.bathrooms>=0) && (this.filtering.bedrooms>=0) && (this.filtering.bathrooms!=="")&& (this.filtering.bedrooms!=="")
        && (Number.isInteger(this.filtering.bathrooms))&& (Number.isInteger(this.filtering.bedrooms))
    }
  },
  methods:{
    dismissFiltering(){
      this.filtering.bathrooms = parseInt(this.$route.query.bathrooms)||0
      this.filtering.bedrooms = parseInt(this.$route.query.bathrooms)||0

      this.toggleFiltering()
    },

    toggleFiltering(){
      this.filteringActive = !this.filteringActive
    },

    setFilters(){
      this.toggleFiltering()
      this.fetchHomes(false)
    },

    toggleType(type){
      if((this.filtering.listingType !== type)){
          this.filtering.listingType = type;
          this.fetchHomes(false)
      }
    },

    debounceInput: debounce(function(){
      if(this.isSearchTextValid){
        this.fetchHomes(false)
      }
    },400),

    setPage(clickedPage){
      this.pagination.page = clickedPage
      this.fetchHomes(true)
    },

    //debounce i toggle predaju false jer se radi o filterima i zelim svaki puta krenuti natrag od prve stranice
    //a set page ce vratiti true. Ako obrisem ovo i npr odem na 35 stranicu renta, nakon toga se toggle na buy a buy ima 30 strana
    //Ako posaljem page = 35, on nece naci niti jednu kucu jer vise nema kuca na 35oj stranici buya. Zato se kod buya po defaultu na apiju predaje 1
    fetchHomes(pagingIncluded){
      let params;
      if(pagingIncluded){
        params={
          searchText: this.filtering.searchText,
          listingTypes: this.filtering.listingType,
          bedrooms: this.filtering.bedrooms,
          bathrooms: this.filtering.bathrooms,
          page: this.pagination.page
        }
      }else{
        params={
          searchText: this.filtering.searchText,
          listingTypes: this.filtering.listingType,
          bedrooms: this.filtering.bedrooms,
          bathrooms: this.filtering.bathrooms
        }
      }

      this.$axios
        .get("https://homehapp-api.jsteam.gaussx.com/api/home", {
          params
        })
        .then(response => {
          //Ne treba mi nova trenutna stranica, ali mi treba novi maksimalni broj stranica.
          this.pagination=response.data.data.pagination

          console.log(response)

          if(this.filtering.listingType===1){
            this.houses = response.data.data.data.filter(house => {return(house.price !== null && house.city !== null)})
          }else{
            this.houses = response.data.data.data.filter(house => {return(house.rentPrice !== null && house.city !== null)})
          }

          this.pushQuery()
        })
        .catch(error => console.log(error))
    },

    pushQuery(){
      let query;
      if(this.filtering.searchText && this.isSearchTextValid){
        {
          query = {
            searchText: this.filtering.searchText,
            page:this.pagination.page,
            listingType: this.filtering.listingType,
            bedrooms: this.filtering.bedrooms,
            bathrooms: this.filtering.bathrooms
          }
        }
      }else{
        query = {
          page:this.pagination.page,
          listingType: this.filtering.listingType,
          bedrooms: this.filtering.bedrooms,
          bathrooms: this.filtering.bathrooms
        }
      }
      this.$router.push({query})
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

  .notification-box{
    @apply bg-indigo-100 w-1/3 mt-20 mx-auto border-t-4 border-indigo-500 rounded-b text-indigo-900 px-4 py-3 shadow-md;
  }

  .search-bar{
    @apply rounded-lg border-2 border-indigo-500 p-3 w-full;
  }

  .btn-open-filters{
    @apply border-2 border-orange-500 my-auto rounded-lg w-12 h-12 bg-orange-500;
  }

  .icon-filter{
    @apply w-6 h-5 mx-auto;
  }

  .filter-bar{
    @apply rounded-lg border-2 border-white p-3 w-full shadow-lg m-2;
  }

  .btn-save-filters{
    @apply block border-2 border-indigo-500 bg-indigo-500 mt-12 mx-auto rounded-lg h-12 w-24;
  }

</style>
