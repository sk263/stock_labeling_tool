<template>
  <div id="app">
    <Labeler />
  </div>
</template>

<script>
import Labeler from './components/Labeler.vue'

export default {
  name: 'App',
  components: {
    Labeler
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
