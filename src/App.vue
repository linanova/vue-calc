<template>
  <v-app id="app">
    <v-container>
      <calculator/>
    </v-container>
  </v-app>
</template>

<script>
import Calculator from './components/Calculator'

export default {
  name: 'app',
  components: {Calculator}
}
</script>

<style>
#app {
  text-align: center;
}
</style>
