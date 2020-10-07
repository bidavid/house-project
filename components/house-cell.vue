<template>
  <div class="p-6 md:w-1/3 sm:w-100%">
    <div class="h-full border-2 border-glitter rounded-lg overflow-hidden hover:shadow-xl">
      <img class="lg:h-48 md:h-36 w-full object-cover object-center" :src="`${imageUrl}${house.cover.id}`" alt="blog">
      <div class="p-3">
        <h2 class="tracking-widest text-sm font-medium text-gray-500 mb-1 truncate">{{ house.address }}, {{ house.city.title}}</h2>
        <hr>
        <h1 v-if="house.listingTypes[0].id===1" class="mt-2 text-lg font-medium text-gray-900">{{formattedPrice}}£</h1>
        <h1 v-else class="mt-2 text-lg font-medium text-gray-900">{{formattedPrice}}£ <span class="text-sm font-medium text-gray-900 mb-3">/ week</span></h1>

        <div class="flex flex-wrap">
          <nuxt-link :to="`/house/${house.id}`">
            <span class="text-purple-600 inline-flex items-center md:mb-2 lg:mb-0">
              Details
            </span>
          </nuxt-link>
          <span class="text-gray-600 mr-3 inline-flex items-center lg:ml-auto md:ml-0 ml-auto leading-none text-sm pr-3 py-1 border-r-2 border-gray-300">
            <img class="w-5 h-5 mr-1" src="/images/bedroom.png">
            <span>{{house.bedrooms}}</span>
          </span>
          <span class="text-gray-600 inline-flex items-center leading-none text-sm">
            <img class="w-5 h-5 mr-1" src="/images/bathroom.png">
            <span>{{house.bathrooms}}</span>
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
      imageUrl:"https://homehapp-api.jsteam.gaussx.com/api/media/"
    }
  },
  props:{
    house: Object
  },
  methods:{

  },
  computed:{
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

</style>
