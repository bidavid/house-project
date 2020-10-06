<template>
  <div class="container mx-auto py-5">
    <div class="flex mb-10 h-12">
      <button @click="toggleType(1)" :class="listingType===1? 'border-purple-600 border-b-4': 'border-gray-350 border-b-2'" class="w-24 py-2 text-xs font-sans">Buy</button>
      <button @click="toggleType(2)" :class="listingType===2? 'border-purple-600 border-b-4': 'border-gray-350 border-b-2'" class="w-24 py-2 text-xs font-sans">Rent</button>
    </div>
    <div v-for="item in homes">
      <div>
        <span>{{item.listingTypes[0].title}}</span>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data(){
    return{
      limitPerPage:20,
      listingType: 1,
      homes: []
    }
  },
  methods:{
    toggleType(type){
      if(this.listingType !== type){
        this.listingType = type;
        this.fetchHomes();
      }else return
    },

    fetchHomes(){
      this.$axios
        .get("https://homehapp-api.jsteam.gaussx.com/api/home", {params:{
            listingTypes:this.listingType,
            limit:this.limitPerPage
          }})
        .then(response=> this.homes = response.data.data.data)
        .catch(error => console.log(error))
    }
  },
  mounted() {
    this.fetchHomes()
  }
}
</script>

<style>
</style>
