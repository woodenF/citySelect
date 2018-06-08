<template>
  <div id="app">
    <city-select></city-select>
  </div>
</template>

<script>
import citySelect from './components/citySelect';
export default {
  name: 'App',
  components: {
    citySelect
  }
};
</script>
<style lang="stylus">
#app
  position fixed
  top 0
  bottom 0
  width 100%
</style>

