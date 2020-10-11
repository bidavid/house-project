<template>
  <div class="p-6 md:w-1/3 sm:w-full">
    <div class="cell-container hover:shadow-lg">
      <img class="lg:h-48 sm:h-36 w-full object-cover object-center" :src="`${imageUrl}`" @error="loadPlaceholder">
      <div class="p-3">
        <h2 class="display-address">{{ house.address }}, {{ house.city.title}}</h2>
        <hr class="border-glitter">
        <h1 v-if="isBuyActive" class="price-title">{{formattedPrice}}£</h1>
        <h1 v-else class="price-title">{{formattedPrice}}£ <span class="text-sm mb-3 text-customRed">/ week</span></h1>

        <div class="flex flex-wrap">
          <nuxt-link :to="`/house/${house.id}`">
            <span class="text-purple-600 font-md text-sm hover:text-customRed md:mb-2 lg:mb-0">
              Details
            </span>
          </nuxt-link>
          <span class="houseroom-info ml-auto mr-3 pr-3 border-r border-glitter">
            <img class="icon" src="/images/bedroom.png">
            <span class="text-gray-500">{{house.bedrooms}}</span>
          </span>
          <span class="houseroom-info">
            <img class="icon" src="/images/bathroom.png">
            <span class="text-gray-500">{{house.bathrooms}}</span>
          </span>
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
  @apply text-glitter inline-flex items-center text-sm;
}

.price-title{
  @apply mt-2 text-lg font-medium text-gray-900;
}

.display-address{
  @apply tracking-widest text-sm font-semibold text-gray-500 mb-1 truncate;
}

</style>
