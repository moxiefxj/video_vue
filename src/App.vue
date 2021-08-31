<template>
  <div id="app">
    <h5-player id="play_wind" :code="code"></h5-player>
    <hik-com id="hik_wind" :code="code"></hik-com>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
import h5Player from './components/h5Play.vue'
import hikCom from './components/hikCom.vue'

export default {
  name: 'App',
  components: {
    HelloWorld,
    h5Player,
    hikCom,
  },
  data() {
    return {
      code: 'xxxxxxxxxx'
    }
  },
}
</script>

<style>
#play_wind, #hik_wind {
  width: 600px;
  height: 300px;
  margin-top: 20px
}
</style>
