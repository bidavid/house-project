<template>
  <div class="w-full my-2 sm:my-0 sm:p-6 md:w-1/3">
    <div class="cell-container hover:shadow-lg">
      <img class="lg:h-48 sm:h-32 w-full object-cover" :src="`${imageUrl}`" @error="loadPlaceholder">
      <div class="p-3">
        <h2 class="display-address">{{ house.address }}, {{ house.city.title}}</h2>
        <hr class="border-glitter">
        <h1 v-if="isBuyActive" class="price-title">{{formattedPrice}}£</h1>
        <h1 v-else class="price-title">{{formattedPrice}}£ <span class="text-sm mb-3 text-customRed">/ week</span></h1>

        <div class="flex flex-wrap">
          <nuxt-link :to="`/house/${house.id}`">
            <div class="text-purple-600 font-md text-sm mb-2 hover:text-customRed sm:mb-0 ">
              Details
            </div>
          </nuxt-link>
          <div class="houseroom-info ml-auto mr-3 pr-3 border-r border-glitter">
            <img class="icon" src="/images/bedroom.png">
            <span class="text-gray-500">{{house.bedrooms}}</span>
          </div>
          <div class="houseroom-info">
            <img class="icon" src="/images/bathroom.png">
            <span class="text-gray-500">{{house.bathrooms}}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {

  data(){
    return{
      imageUrl:`https://homehapp-api.jsteam.gaussx.com/api/media/${this.house.cover.id}/small`
    }
  },

  props:{
    house: Object
  },

  methods:{
    loadPlaceholder(){
      this.imageUrl = `/images/placeholder.png`
    }
  },

  computed:{
    isBuyActive(){
      return this.house.listingTypes[0].id===1
    },
    formattedPrice(){
      const formatter =  new Intl.NumberFormat('en-US', {
        minimumFractionDigits: 0
      });

      if(this.house.listingTypes[0].id===1){
        return formatter.format(this.house.price);
      }else{
        return formatter.format(this.house.rentPrice);
      }
    }
  }
}

</script>

<style>

.cell-container{
 @apply h-full border-2 border-purple-100 rounded-lg overflow-hidden bg-purple-100;
}

.icon{
  @apply w-5 h-5 mr-1;
}

.houseroom-info{
  @apply text-glitter flex items-center text-sm;
}

.price-title{
  @apply mt-2 text-lg font-medium text-gray-900;
}

.display-address{
  @apply tracking-widest text-sm text-gray-500 mb-1 truncate;
}

</style>
