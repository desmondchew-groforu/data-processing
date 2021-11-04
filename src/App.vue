<template>
  <div id="app">
    <img alt="Groforu logo" src="./assets/groforuLogo.png" height="160px" width="160px">
    <PhoneNumberFormatter/>
  </div>
</template>

<script>
import PhoneNumberFormatter from './components/PhoneNumberFormatter.vue'

export default {
  name: 'App',
  components: {
    PhoneNumberFormatter
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  /* text-align: center; */
  color: #2c3e50;
  margin-top: 60px;
}
</style>
