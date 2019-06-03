<template>
  <div id="app">
    <Player/>
  </div>
</template>

<script lang="ts">
  import {Component, Vue} from 'vue-property-decorator';
  import Player from '@/components/Player.vue';

  @Component({
    components: {
      Player,
    },
  })
  export default class App extends Vue {}
</script>

<style lang="scss">

  *
  {
    margin: 0;
    padding: 0;
  }

  img
  {
    vertical-align: middle;
  }

  html, body
  {
    min-height: 100vh;
  }

  body
  {
    background: radial-gradient(ellipse at bottom right, #454651 0%, #50526b 100%);
  }

  #app
  {
    font-family:             'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing:  antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align:              center;
    color:                   #dde;
    height: 100%;
  }

</style>
