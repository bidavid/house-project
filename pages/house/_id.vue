<template>
<div class="px-5 py-24 mx-auto">
  <div v-if="houseInfo && houseInfo.basicInfo && houseInfo.agencyInfo.agencyTitle" class="lg:w-4/6 mx-auto">
    <img class="h-64 w-full rounded-lg object-cover object-center overflow-hidden" :src="`${houseInfo.imageUrl}`" @error="loadPlaceholder">
    <div class="p-3">
      <h2 class="display-address">{{ houseInfo.basicInfo.address }}, {{ houseInfo.basicInfo.city.title}}</h2>
      <hr class="border-glitter">
      <div class="flex flex-wrap items-center">
        <h1 v-if="isBuyActive" class="price-title">{{formattedPrice}}£</h1>
        <h1 v-else class="price-title">{{formattedPrice}}£ <span class="text-sm mb-3 text-customRed">/ week</span></h1>
        <span class="houseroom-info ml-auto mr-3 pr-3 border-r border-glitter">
          <img class="icon" src="/images/bedroom.png">
          <span class="text-gray-500">{{houseInfo.basicInfo.bedrooms}}</span>
        </span>
        <span class="houseroom-info">
          <img class="icon" src="/images/bathroom.png">
          <span class="text-gray-500">{{houseInfo.basicInfo.bathrooms}}</span>
        </span>
      </div>
    </div>
    <div class="flex flex-col sm:flex-row mt-4">
      <div class="sm:w-1/3 text-center sm:pr-8 sm:py-8">
        <div class="w-20 h-20 rounded-full inline-flex items-center justify-center bg-gray-200 text-gray-400">
          <img class="w-20 h-20 rounded-full object-cover object-center overflow-hidden" :src="`${houseInfo.imageUrl}`" @error="loadPlaceholder">
        </div>
        <div class="flex flex-col items-center text-center justify-center">
          <h2 class="font-medium title-font mt-4 text-gray-900 text-lg">{{houseInfo.agencyInfo.agencyTitle}}</h2>
          <div class="w-12 h-1 bg-indigo-500 rounded mt-2 mb-4"></div>
          <p class="text-base text-gray-600">Raclette knausgaard hella meggs normcore williamsburg enamel pin sartorial venmo tbh hot chicken gentrify portland.</p>
        </div>
      </div>
      <div class="sm:w-2/3 sm:pl-8 sm:py-4 sm:border-l border-gray-300 sm:border-t-0 border-t mt-4 pt-4 sm:mt-0 text-center sm:text-left">
        <p class="leading-relaxed text-lg mb-4 text-justify">{{houseInfo.basicInfo.description}}</p>
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
  data(){
    return{
      idForDisplay:parseInt(this.$route.params.id),
      houseInfo:{
        agencyInfo:{
          agencyTitle:"",
          agencyImageUrl:""
        },
        basicInfo:{},
        imageUrl:""
      }
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
      if(this.houseInfo.basicInfo.listingTypes[0].id===1){
        return formatter.format(this.houseInfo.basicInfo.price);
      }else{
        return formatter.format(this.houseInfo.basicInfo.rentPrice);
      }
    }
  },

  methods:{
    loadPlaceholder(){
      this.houseInfo.imageUrl = `/images/placeholder.png`
    },

    async fetchHomeInfo(){
      let response = await this.$axios.get(`https://homehapp-api.jsteam.gaussx.com/api/home/${this.idForDisplay}`)
      console.log(response)
      this.houseInfo.basicInfo=response.data.data
      this.houseInfo.imageUrl = `https://homehapp-api.jsteam.gaussx.com/api/media/${this.houseInfo.basicInfo.cover.id}`

    },
    async fetchAgencyTitle(){
      let response = await this.$axios.get(`https://homehapp-api.jsteam.gaussx.com/api/agency/${this.houseInfo.basicInfo.agency_id}`)
      console.log(response)
      this.houseInfo.agencyInfo.agencyTitle = response.data.data.title
      this.houseInfo.agencyInfo.agencyImageUrl = `https://homehapp-api.jsteam.gaussx.com/api/media/${response.data.data.cover.id}`
    }
  },
  async mounted(){
    try{
      await this.fetchHomeInfo()
      await this.fetchAgencyTitle()
    }catch(error){
      console.log(error)
    }
  }
}
</script>

<style>
.apology-box{
  @apply bg-purple-100 w-1/3 mt-20 mx-auto border-t-4 border-purple-500 rounded-b text-purple-900 px-4 py-3 shadow-md;
}

.display-address{
  @apply tracking-widest text-sm font-semibold text-gray-500 mb-1 truncate;
}
</style>
