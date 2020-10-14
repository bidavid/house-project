<template>
  <div class="house-main-container">
    <div class="house-content-container">
      <img class="house-image-cover" :src="`${imageUrl}`" @error="loadPlaceholder">
      <div class="house-data-container">
        <h2 class="house-display-address">{{house.address}}, {{ house.city.title}}</h2>
        <hr>
        <h2 v-if="isBuyActive" class="house-price">{{formattedPrice}}£</h2>
        <h2 v-else class="house-price">{{formattedPrice}}£<span class="weekly-text"> / week</span></h2>
        <div class="house-content-footer">
            <div>
              <nuxt-link :to="`../house/${house.id}`">
                <span class="house-details-link">
                  Details
                </span>
              </nuxt-link>
            </div>
            <div style="display: flex">
              <div class="bedroom-container">
                <img class="houseroom-icon" src="/images/bedroom.png">
                <span class="houseroom-number">{{house.bedrooms}}</span>
              </div>
              <div class="bathroom-container">
                <img class="houseroom-icon" src="/images/bathroom.png">
                <span class="houseroom-number">{{house.bathrooms}}</span>
              </div>
            </div>
        </div>
      </div>
    </div>
  </div>
</template>



<script>
import houseCell from '~/components/house-cell.vue'

export default {
  name: "house-cell",
  extends:houseCell
}
</script>

<style scoped>
.house-main-container{
  width: 33.33%;
  padding: 2%;
}

@media only screen and (max-width: 600px) {
  .house-main-container {
    width: 100%;
  }
}

.house-content-container{
  border-radius: 0.5rem;
  width: 100%;
  background-color: rgba(242, 237, 251,0.7);
  border: 1px solid rgba(242, 237, 251,0.7);
  overflow: hidden;
}

.house-content-container:hover{
  box-shadow: 10px 10px 15px rgba(0, 0, 0, 0.1), 0 4px 6px rgba(0, 0, 0, 0.05);
}

.house-image-cover{
  object-fit: cover;
  object-position: center;
  width: 100%;
}

@media only screen and (min-width: 600px) {
  .house-image-cover {
    height: 8rem;
  }
}

@media only screen and (min-width: 1024px) {
  .house-image-cover {
    height: 12rem;
  }
}

.house-data-container{
  padding-top: 3%;
  padding-bottom: 3%;
  padding-left: 2%;
  padding-right: 2%;
}

.house-display-address{
  font-size: 0.875rem;
  font-weight: 400;
  color: #BABABA;
  letter-spacing: 0.1em;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.house-price{
  font-weight: 600;
}

.weekly-text{
  color: #D81B60;
  font-size: small;
  font-weight: bold;
}

.house-content-footer{
  display: flex;
  flex-wrap:wrap;
  justify-content: space-between;
}

.house-details-link{
  font-size: small;
  color: mediumpurple;
}

.house-details-link:hover{
  color: #D81B60;
}

.houseroom-icon{
  height: 22px;
  width: 22px;
}

.houseroom-number{
  font-size: small;
  color: #BABABA;
}

.bedroom-container{
  display:flex; padding-right:5px; border-right: 1px solid #E6E8FA
}

.bathroom-container{
  display:flex; padding-left:5px
}
</style>
