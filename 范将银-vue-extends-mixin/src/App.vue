<template>
  <div id="app">
    <coffee-cat class="cat_card"></coffee-cat>
    <persian-cat class="cat_card"></persian-cat>
    <short-hair-cat class="cat_card"></short-hair-cat>
    <house-cat class="cat_card"></house-cat>
  </div>
</template>

<script>
import CoffeeCat from '@/components/cat/CoffeeCat'
import HouseCat from '@/components/cat/HouseCat'
import PersianCat from '@/components/cat/PersianCat'
import ShortHairCat from '@/components/cat/ShortHairCat'

export default {
  name: 'app',
  components: {
    CoffeeCat,
    HouseCat,
    PersianCat,
    ShortHairCat
  }
}
</script>

<style lang="scss" scoped>
  .cat_card{
    float: left;
    padding-right: 8px;
    padding-bottom: 8px;
  }
</style>
