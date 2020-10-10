<template>
  <div class="container w-1/3 flex justify-center mt-5">

    <div v-if="pagination.page > 1" @click="emitClick(1)" class="paging-arrow-cell">
      <img class="icon" src="/images/navigate_first.png">
    </div>

    <div v-if="pagination.page > 1" @click="emitClick(pagination.page - 1)" class="paging-arrow-cell">
      <img class="icon" src="/images/navigate_back.png">
    </div>

    <div @click="emitClick(startingCellNumber)" :class="pagination.page===startingCellNumber? 'active-cell': 'inactive-cell'" class="paging-cell hover:text-customRed">
      {{startingCellNumber}}
    </div>

    <div v-if="middleCellNumber" @click="emitClick(middleCellNumber)" :class="pagination.page===middleCellNumber? 'active-cell': 'inactive-cell'" class="paging-cell hover:text-customRed">
      {{middleCellNumber}}
    </div>

    <div v-if="endingCellNumber" @click="emitClick(endingCellNumber)" :class="pagination.page===endingCellNumber? 'active-cell': 'inactive-cell'" class="paging-cell hover:text-customRed">
      {{endingCellNumber}}
    </div>

    <div v-if="pagination.page < pagination.lastPage" @click="emitClick(pagination.page + 1)" class="paging-arrow-cell">
      <img class="icon" src="/images/navigate_next.png">
    </div>

    <div v-if="pagination.page < pagination.lastPage" @click="emitClick(pagination.lastPage)" class="paging-arrow-cell">
      <img class="icon" src="/images/navigate_last.png">
    </div>

  </div>

</template>

<script>
export default {
  props:{
    pagination:Object
  },
  methods:{
    emitClick(clickedPageNumber){
      this.$emit('pageNumberClicked', clickedPageNumber)
    }

  },
  computed:{
    startingCellNumber(){
      if(this.pagination.page!==1 && this.pagination.page!==this.pagination.lastPage){

        return this.pagination.page - 1

      }else if(this.pagination.page ===1){

        return this.pagination.page

      }else{
        return this.pagination.lastPage - 2
      }
    },

    middleCellNumber(){
      if(this.pagination.lastPage < 2){

        return false

      }else{
        if(this.pagination.page!==1 && this.pagination.page!==this.pagination.lastPage){

          return this.pagination.page

        }else if(this.pagination.page === 1){

          return this.pagination.page + 1

        }else{
          return this.pagination.lastPage - 1
        }
      }
    },

    endingCellNumber(){
      if(this.pagination.lastPage<3){

        return false

      }else{
        if(this.pagination.page!==1 && this.pagination.page!==this.pagination.lastPage){

          return this.pagination.page + 1

        }else if(this.pagination.page === 1){

          return this.pagination.page + 2

        }else{
          return this.pagination.lastPage
        }
      }

    }
  }
}
</script>

<style scoped>

.icon{
  @apply w-5 h-5 mr-0 my-auto;
}

.paging-arrow-cell{
  @apply pt-3 cursor-pointer rounded-lg mx-2;
}

.paging-cell{
 @apply inline-block px-4 py-2 border text-sm font-medium cursor-pointer rounded-lg mx-1;
}

.active-cell{
  @apply bg-purple-600 border-purple-600 text-white;
}

.inactive-cell{
  @apply border-glitter  text-purple-300;
}


</style>
