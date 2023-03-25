<template>
  <div>
    <h1>При нажатии на время можете его менять</h1>
    <div>
      <v-timer />
    </div>
  </div>
</template>
<script>
import index from '../components/index.vue';
export default {
  components: {
    'v-timer': index
  }
}
</script>
<style lang="css">
h1 {
  text-align: center;
  padding: 0 0 50px 0;
}
</style>