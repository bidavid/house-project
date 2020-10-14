<template>
  <div class="main-container">

    <div class="heading-container">
      <div class="toggle-buttons-container">
        <button @click="toggleType(1)" class="toggle-button" :disabled="!isSearchTextValid"  :class="[{'toggle-button-active':isBuyActive}, {'toggle-button-disabled':!isSearchTextValid}]">Buy</button>
        <button @click="toggleType(2)" class="toggle-button" :disabled="!isSearchTextValid" :class="[{'toggle-button-active':isRentActive}, {'toggle-button-disabled':!isSearchTextValid}]">Rent</button>
      </div>
      <div class="filters-container">
        <input @input="debounceInput" v-model="filtering.searchText" class="filter-search-bar" type="text" placeholder="Enter keyword...">
        <button @click="toggleFiltering" :disabled="!isSearchTextValid" :class="isSearchTextValid? 'open-filters-button-active':'open-filters-button-inactive'">Filters</button>
      </div>
    </div>

    <div class="content-container" v-if="houses.length > 0 ">
      <div class="houses-container">
        <house-cell v-for="item in houses" :key="item.id" :house="item"></house-cell>
      </div>

      <custom-pagination v-if="isSearchTextValid" @pageNumberClicked="setPage" class="mx-auto" :pagination="pagination"> </custom-pagination>
    </div>

    <div v-if="filteringActive" class="filtering-window-container">
      <div @click="dismissFiltering" class=".filtering-window-transparent-css" >
      </div>
      <div class=".filtering-window-opaque-css">
        <div class="h-auto w-3/5 mx-auto my-24">
          <div class="mb-3">
            <span class="filtering-title-span">Min. bedrooms: </span>
            <input class="filtering-content-input focus:outline-none" v-model.number="filtering.bedrooms" type="number" min="0">
          </div>
          <div class="mb-3">
            <span class="filtering-title-span">Min. bathrooms: </span>
            <input class="filtering-content-input focus:outline-none" v-model.number="filtering.bathrooms" type="number" min="0">
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
import index from '~/pages/index.vue'
import houseCell from '~/components/test/house-cell.vue'

export default {
  name:"index",
  extends: index,
  components:{
    'house-cell':houseCell
  }
}
</script>

<style>
.main-container{
  height:100%;
  min-width:100%;
}

.heading-container{
  height:100%;
  width:100%;
  padding-top: 3%;
  padding-left: 5%;
  padding-right: 5%;
  margin:auto;
  display: flex;
  flex-wrap: wrap;
  flex-direction: row;
  justify-content: space-between;
}

@media only screen and (max-width: 600px) {
  .heading-container {
    width:100%;
    padding-left: 0;
    padding-right: 0;
  }
}

.toggle-buttons-container{
  width: 250px;
  display: flex;
}

@media only screen and (max-width: 600px) {
  .toggle-buttons-container {
    width:100%;
  }
}

.toggle-button{
  width: 50%;
  border-bottom:3px solid lightgray;
  text-transform: uppercase;
  font-size: small;
  font-weight: bold;
  color: lightgray
}

.toggle-button-active{
  color: mediumpurple;
  border-bottom:3px solid mediumpurple;
}

.toggle-button-disabled{
  cursor: default;
  opacity: 0.5;
}

.toggle-button:focus{
  outline: 0;
}

.filters-container{
  width: 500px;
  display: flex;
  justify-content: flex-end;
}

@media only screen and (max-width: 600px) {
  .filters-container {
    padding-top: 15px;
    padding-left: 10px;
    padding-right: 10px;
    padding-bottom: 15px;
  }
}

.filter-search-bar{
  width: 70%;
  border-radius: 0.5rem;
  padding:10px;
  border: 1px solid #e6e8fa;
  color:mediumpurple;
  margin-right: 15px;
}

.filter-search-bar:focus{
  outline: 0;
}

.open-filters-button-active{
  border-radius: 0.5rem;
  border: 5px solid mediumpurple;
  background-color: mediumpurple;
  color: white;
  font-weight: 1.6;
  font-size: small;
  letter-spacing: 0.1em;
  width: 80px;
  text-transform: uppercase;
}

.open-filters-button-active:focus{
  outline: 0;
}

.open-filters-button-active:hover{
  background-color: #D81B60;
  border: 5px solid #D81B60;
}

.open-filters-button-inactive{
  border-radius: 0.5rem;
  border: 5px solid mediumpurple;
  background-color: mediumpurple;
  color: white;
  font-weight: bold;
  font-size: small;
  width: 80px;
  text-transform: uppercase;
  opacity: 0.5;
  cursor: default;
}

.open-filters-button-inactive:focus{
  outline: 0;
}

.content-container{
  max-width: 100%;
  min-width: 100%;
  padding-left: 3%;
  padding-right: 3%;
  margin-top: 2%;
  padding-bottom: 2%;
}

@media only screen and (max-width: 600px) {
  .content-container {
    padding-left: 5px;
    padding-right: 5px;
    padding-bottom: 4%;
  }
}

.houses-container{
  display: flex;
  flex-wrap:wrap;
}

.filtering-window-container{
  height: 100%;
  width: 100%;
  display: flex;
  position: absolute;
  top:0;
  left: 0;
}

.filtering-window-transparent-css{

}
.filtering-window-opaque-css{

}

</style>
