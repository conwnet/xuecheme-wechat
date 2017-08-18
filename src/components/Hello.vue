<template>
  <div class="hello">
    <mt-button>123</mt-button>
    <mt-swipe :auto="4000">
      <mt-swipe-item>1</mt-swipe-item>
      <mt-swipe-item>2</mt-swipe-item>
      <mt-swipe-item>3</mt-swipe-item>
    </mt-swipe>
    123
  </div>
</template>

<script>

/**
import { Swipe, SwipeItem } from 'mint-ui'

Vue.component('mt-swipe', Swipe)
Vue.component('mt-swipe-item', SwipeItem)
*/

export default {
  name: 'hello',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App'
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
