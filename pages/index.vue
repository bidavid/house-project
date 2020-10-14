<template>
  <div class="sm:py-5 sm:px-5 py-2 px-2">
    <div class="flex sm:py-5 sm:px-5 flex-wrap w-full justify-center">
        <div class="sm:flex ms:mr-auto py-4">
          <button :disabled="!isSearchTextValid" @click="toggleType(1)" :class="[{'toggle-active':isBuyActive},{'toggle-inactive':isRentActive},{'cursor-default opacity-50':!isSearchTextValid}]" class="btn-toggle focus:outline-none">Buy</button>
          <button :disabled="!isSearchTextValid" @click="toggleType(2)" :class="[{'toggle-active':isRentActive},{'toggle-inactive':isBuyActive}, {'cursor-default opacity-50':!isSearchTextValid}]" class="btn-toggle focus:outline-none">Rent</button>
        </div>

        <div v-if="!isSearchTextValid" class="error-box">
          <strong class="align-middle"> Invalid input!</strong>
          <span class="align-middle"> Looks like you entered invalid character or whitespace. Filtering, paging and toggling disabled.</span>
        </div>

        <div class="inline-block md:ml-auto my-3 md:w-1/4">
          <input @input="debounceInput" v-model="filtering.searchText" class="search-bar focus:outline-none placeholder-gray-500" placeholder="Enter keyword.." type="text" >
        </div>

        <button @click="toggleFiltering" :disabled="!isSearchTextValid" :class="isSearchTextValid? 'opacity-100 hover:border-customRed hover:bg-customRed hover:shadow-xl':'opacity-50 cursor-default'" class="btn-open-filters sm:ml-5 focus:outline-none">
          <span class="text-white uppercase text-sm font-sans tracking-widest">Filters</span>
        </button>

    </div>

    <div v-if="houses.length > 0 ">
      <div class="flex flex-wrap">
        <house-cell v-for="item in houses" :key="item.id" :house="item"></house-cell>
      </div>

      <custom-pagination v-if="isSearchTextValid" @pageNumberClicked="setPage" class="mx-auto" :pagination="pagination"> </custom-pagination>
    </div>

    <div v-else class="notification-box sm:w-1/3">
      <div class="flex">
        <div class="py-1">
          <svg class="fill-current h-6 w-6 text-customRed mr-4" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20"><path d="M2.93 17.07A10 10 0 1 1 17.07 2.93 10 10 0 0 1 2.93 17.07zm12.73-1.41A8 8 0 1 0 4.34 4.34a8 8 0 0 0 11.32 11.32zM9 11V9h2v6H9v-4zm0-6h2v2H9V5z"/></svg>
        </div>
        <div>
          <p class="font-bold">0 objects fetched.</p>
          <p class="text-sm">No object matches your keyword.</p>
        </div>
      </div>
    </div>

    <div v-if="filteringActive" class="filtering-window-parent">
      <div @click="dismissFiltering" class="filtering-window-transparent sm:w-2/3">
      </div>
      <div class="filtering-window-opaque sm:w-1/3">
        <div class="h-auto w-3/5 mx-auto my-24">
          <div class="mb-3">
            <label for="bedroomsInputFilter" class="filtering-title-span">Min. bedrooms: </label>
            <input id="bedroomsInputFilter" class="filtering-content-input focus:outline-none" v-model.number="filtering.bedrooms" type="number" min="0">
          </div>
          <div class="mb-3">
            <label for="bathroomsInputFilter" class="filtering-title-span">Min. bathrooms: </label>
            <input id="bathroomsInputFilter" class="filtering-content-input focus:outline-none" v-model.number="filtering.bathrooms" type="number" min="0">
          </div>
          <div class="mt-6">
            <button @click="saveFilters" :disabled="!areFiltersValid" :class="areFiltersValid? 'opacity-100 hover:border-customRed hover:bg-customRed hover:shadow-xl':'opacity-50 cursor-default'" class="btns-filtering focus:outline-none">
              Save
            </button>

            <button @click="resetFilters" class="btns-filtering focus:outline-none hover:border-customRed hover:bg-customRed hover:shadow-xl">
              Reset
            </button>
          </div>
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
        bedrooms:parseInt(this.$route.query.bedrooms)||0,
        limit:9
      },
      pagination:{
        page: parseInt(this.$route.query.page) || 1
      },
    }
  },
  watch:{
    '$route.query'(query){
      //Sada opet moram iscitati postaviti sve varijable iz queryja zato jer mi inace nece raditi back button na browseru,
      //bez ovoga bi sve radilo ali bez back button ne
      this.readQueryParams()

      console.log("query promijenjen!")
      if(this.isSearchTextValid){
        this.fetchHomes()
      }
    }
  },
  computed:{
    isBuyActive(){
      return this.filtering.listingType===1
    },
    isRentActive(){
      return this.filtering.listingType===2
    },
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
      this.filtering.bedrooms = parseInt(this.$route.query.bedrooms)||0

      this.toggleFiltering()
    },

    toggleFiltering(){
      this.filteringActive = !this.filteringActive
    },

    saveFilters(){
      this.toggleFiltering()
      this.pushQuery(false)
    },

    resetFilters(){
      this.toggleFiltering()

      if(this.filtering.bedrooms!==0 || this.filtering.bathrooms !==0){
        this.filtering.bedrooms = 0
        this.filtering.bathrooms = 0
        this.pushQuery(false)
      }
    },

    toggleType(type){
      if((this.filtering.listingType !== type)){
          this.filtering.listingType = type;
          this.pushQuery(false)
      }
    },

    debounceInput: debounce(function(){
      if(this.isSearchTextValid){
        this.pushQuery(false)
      }
    },400),

    setPage(clickedPage){
      this.pagination.page = clickedPage
      this.pushQuery(true)
    },

    readQueryParams(){
      this.filtering.searchText= this.$route.query.searchText || ""
      this.filtering.listingType= parseInt(this.$route.query.listingType) || 1
      this.filtering.bathrooms= parseInt(this.$route.query.bathrooms)||0
      this.filtering.bedrooms= parseInt(this.$route.query.bedrooms)||0
      this.pagination.page= parseInt(this.$route.query.page) || 1
    },

    fetchHomes(){
      let params={
          searchText: this.filtering.searchText,
          listingTypes: this.filtering.listingType,
          bedrooms: this.filtering.bedrooms,
          bathrooms: this.filtering.bathrooms,
          limit:this.filtering.limit,
          page: this.pagination.page
        }

      this.$axios
        .get("https://homehapp-api.jsteam.gaussx.com/api/home", {
          params
        })
        .then(response => {
          //nakon fetchanja nove stranice mi ne treba nova trenutna stranica, ali mi treba novi maksimalni broj stranica.
          this.pagination=response.data.data.pagination

          //console.log(response)

          if(this.filtering.listingType===1){
            this.houses = response.data.data.data.filter(house => {
              return(house.id !== null && house.price !== null && house.city !== null)})
          }else{
            this.houses = response.data.data.data.filter(house => {
              return(house.id !== null && house.rentPrice !== null && house.city !== null)})
          }
        })
        .catch(error => console.log(error))
    },

    //debounce, toggle i setFilters predaju false jer se radi o filterima i zelim svaki puta krenuti natrag od prve stranice
    //a set page ce poslati true. Ako obrisem ovo i npr odem na 35 stranicu renta, nakon toga se toggle na buy a buy ima 30 strana
    //Ako posaljem page = 35, on nece naci niti jednu kucu jer vise nema kuca na 35oj stranici buya. Zato se kod buya po defaultu na apiju predaje 1
    pushQuery(includePaging){
      let query;
      //if else omogucava da se searchText ne zapisuje u query ako nije validan ili ako je prazan
      if(this.filtering.searchText && this.isSearchTextValid){
        query = {
          searchText: this.filtering.searchText,
          page:includePaging? this.pagination.page : 1,
          listingType: this.filtering.listingType,
          bedrooms: this.filtering.bedrooms,
          bathrooms: this.filtering.bathrooms,
        }
      }else{
        query = {
          page:includePaging? this.pagination.page : 1,
          listingType: this.filtering.listingType,
          bedrooms: this.filtering.bedrooms,
          bathrooms: this.filtering.bathrooms,
        }
      }

      this.$router.push({query})
    }
  },

  mounted() {
    this.fetchHomes()
  }
}
</script>

<style>
  .btn-toggle{
   @apply text-xs font-bold font-sans uppercase w-24 border-b-4;
  }

  .toggle-active{
    @apply border-purple-500 text-purple-500;
  }

  .toggle-inactive{
    @apply border-gray-300 text-gray-300;
  }

  .error-box{
    @apply tracking-widest border border-red-400 text-red-700 rounded mx-auto text-xs font-semibold p-3 my-3;
  }

  .notification-box{
    @apply bg-purple-100 mt-20 mx-auto border-t-4 border-purple-500 rounded-b text-purple-900 px-4 py-3 shadow-md;
  }

  .search-bar{
    @apply rounded-lg border border-glitter p-3 w-full;
  }

  .btn-open-filters{
    @apply border-2 border-purple-500 my-auto rounded-lg w-24 h-12 bg-purple-500 ml-2;
  }

  .btns-filtering{
    @apply inline-block border-2 border-purple-500 bg-purple-500 mr-10 rounded-lg h-12 w-24 text-white uppercase text-sm font-semibold font-sans tracking-widest mt-2;
  }

  .filtering-window-parent{
    @apply absolute fixed top-0 left-0 w-full h-screen flex flex-wrap;
  }

  .filtering-window-transparent{
    @apply h-screen w-1/3 opacity-50 m-0 bg-gray-500;
  }

  .filtering-window-opaque{
    @apply h-screen w-2/3 bg-white flex place-content-center;
  }

  .filtering-title-span{
    @apply text-purple-400 tracking-widest font-sans text-lg block mt-4;
  }

  .filtering-content-input{
    @apply rounded-lg border border-glitter p-3 w-1/3 shadow-lg m-2;
  }


</style>
