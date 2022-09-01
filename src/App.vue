<template>
  <div class="app">
    <Building />
  </div>
</template>

<script>
import Building from './components/Building.vue'

export default {
  name: 'App',
  components: {
    Building
  }
}
</script>

<style lang="scss">
.app {
  width: 600px;
  margin: 50px auto 0 auto;
}
</style>
