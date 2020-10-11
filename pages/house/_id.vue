<template>
<div class="px-5 py-20 mx-auto">
  <div v-if="houseInfo && houseInfo.basicInfo && houseInfo.agencyInfo" class="lg:w-4/6 mx-auto">
    <img class="h-64 w-full rounded-lg object-cover object-center overflow-hidden" :src="`${houseInfo.basicInfo.imageUrl}`" @error="loadHousePlaceholder">
    <div class="p-3">

      <h2 class="house-address-title">{{ houseInfo.basicInfo.address }}, {{ houseInfo.basicInfo.city.title}}</h2>
      <hr class="border-glitter">

      <div class="flex flex-wrap mt-2">
        <h1 v-if="isBuyActive" class="price-title sm:mb-0">{{formattedPrice}}£</h1>
        <h1 v-else class="price-title sm:mb-0">{{formattedPrice}}£ <span class="text-sm mb-3 text-customRed">/ week</span></h1>
        <span class="house-houseroom-info sm:mr-3 pr-3 sm:ml-auto sm:border-r border-glitter">
          <img class="houseroom-icon" src="/images/bedroom.png">
          <span class="text-gray-500">{{houseInfo.basicInfo.bedrooms}}</span>
        </span>
        <span class="house-houseroom-info sm:mx-0">
          <img class="houseroom-icon" src="/images/bathroom.png">
          <span class="text-gray-500">{{houseInfo.basicInfo.bathrooms}}</span>
        </span>
      </div>
    </div>

    <div class="flex flex-col sm:flex-row mt-4 p-6 border border-purple-100 shadow rounded-lg bg-purple-100">
      <div class="sm:w-1/5 sm:pr-6 overflow-hidden">
        <p class="text-lg mb-2 text-left font-medium text-gray-900">Features:</p>
        <ul v-if="houseInfo.basicInfo.features.length > 0" class="list-disc flex flex-col pl-6 justify-center flex-wrap">
          <li v-for="(feature,index) in houseInfo.basicInfo.features" :key="index" class="mr-auto text-gray-600" >{{feature.title}}</li>
        </ul>

        <p v-else class="w-full rounded-full mt-6 text-center mx-auto bg-purple-600 text-white font-semibold uppercase tracking-widest text-xs"> Not available </p>
      </div>

      <div class="sm:w-4/5 sm:pl-6 sm:border-l border-gray-300 sm:border-t-0 border-t mt-4 sm:mt-0 text-center sm:text-left">
        <p class="text-lg mb-2 text-left font-medium text-gray-900">Description:</p>
        <p class="text-lg text-justify text-gray-600">{{houseInfo.basicInfo.description}}</p>
      </div>
    </div>

    <div class="w-full mt-10">
      <img class="w-20 h-20 rounded-full mx-auto object-cover object-center overflow-hidden" :src="`${houseInfo.agencyInfo.imageUrl}`" @error="loadAgencyPlaceholder">
      <div class="flex flex-col items-center text-center w-2/5 mx-auto">
        <h2 class="font-medium title-font mt-4 text-gray-900 text-lg">{{houseInfo.agencyInfo.title}}</h2>
        <div class="w-full h-1 bg-purple-500 rounded mt-2 mb-4"></div>
        <p class="text-sm font-semibold text-gray-900">{{houseInfo.agencyInfo.specialities}}</p>
        <p class="text-xs text-purple-400 mt-3 cursor-pointer italic">{{houseInfo.agencyInfo.website}}</p>
      </div>
    </div>
  </div>

  <div v-else class="apology-box">
    <div class="flex">
      <div class="py-1">
        <svg class="fill-current h-6 w-6 text-customRed mr-4" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20"><path d="M2.93 17.07A10 10 0 1 1 17.07 2.93 10 10 0 0 1 2.93 17.07zm12.73-1.41A8 8 0 1 0 4.34 4.34a8 8 0 0 0 11.32 11.32zM9 11V9h2v6H9v-4zm0-6h2v2H9V5z"/></svg>
      </div>
      <div>
        <p class="font-bold">We are sorry, but something went wrong with your request.</p>
        <p class="text-sm">Please go back and try again.</p>
      </div>
    </div>
  </div>
</div>

</template>

<script>

export default {
  async asyncData({params, $axios}){
    try{
      let houseInfo = {}
      let response = await $axios.get(`https://homehapp-api.jsteam.gaussx.com/api/home/${parseInt(params.id)}`)
      houseInfo.basicInfo = response.data.data
      houseInfo.basicInfo.imageUrl = `https://homehapp-api.jsteam.gaussx.com/api/media/${response.data.data.cover.id}`

      response = await $axios.get(`https://homehapp-api.jsteam.gaussx.com/api/agency/${houseInfo.basicInfo.agency_id}`)
      houseInfo.agencyInfo = response.data.data
      houseInfo.agencyInfo.imageUrl = `https://homehapp-api.jsteam.gaussx.com/api/media/${response.data.data.cover.id}`

      return{houseInfo:houseInfo}

    }catch(error){
      console.log(error)
    }
  },

  computed:{
    isBuyActive(){
      return this.houseInfo.basicInfo.listingTypes[0].id===1
    },

    formattedPrice(){
      const formatter =  new Intl.NumberFormat('en-US', {
        minimumFractionDigits: 0
      });
      if(this.isBuyActive){
        return formatter.format(this.houseInfo.basicInfo.price);
      }else{
        return formatter.format(this.houseInfo.basicInfo.rentPrice);
      }
    }
  },

  methods:{
    loadHousePlaceholder(){
      this.houseInfo.basicInfo.imageUrl = `/images/placeholder.png`
    },
    loadAgencyPlaceholder(){
      this.houseInfo.agencyInfo.imageUrl = `/images/question_mark.png`
    }
  }
}
</script>

<style>
.house-address-title{
  @apply tracking-widest text-lg font-semibold text-purple-600 mt-2 truncate;
}

.house-houseroom-info{
  @apply text-glitter inline-flex items-center text-sm mx-auto;
}

.houseroom-icon{
  @apply mr-1;
}

.price-title{
  @apply mt-2 text-lg font-medium text-gray-900 mb-1;
}

.apology-box{
  @apply bg-purple-100 w-1/3 mt-20 mx-auto border-t-4 border-purple-500 rounded-b text-purple-900 px-4 py-3 shadow-md;
}


</style>
