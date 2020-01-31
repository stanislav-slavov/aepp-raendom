<template>
  <div id="app">
    <TipsList />
  </div>
</template>

<script>
import TipsList from './components/TipsList.vue'

export default {
  name: 'app',
  components: {
      TipsList
  },
  created () {}
}
</script>

<style>
#app {
  font-family: 'Roboto', Helvetica, Arial, sans-serif;
  background-color: #26221E;
}
</style>
