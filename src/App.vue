<template>
  <div id="app">
    <SVGGen :width="30" :height="20" :maxFiguresAmt="2" :availableFigures="['rect', 'circle']" :defaultFigures="[]" />
  </div>
</template>

<script>
import SVGGen from './components/SVGGen.vue'

export default {
  name: 'App',
  components: {
    SVGGen
  }
}
</script>

<style lang="scss">
#app {
  font-family: Arial, sans-serif;
}
</style>
