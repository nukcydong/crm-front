<template>
  <ul class="list">
    <slot/>
  </ul>
</template>
<script>
  import Vue from 'vue';
  import {ContainerMixin} from './vue-slicksort.cjs.js';

  export default {
    name: "dragList",
    mixins: [ContainerMixin],
  }
</script>

<style scoped lang="scss" type="text/scss">
.list{
  padding: 0px;
  margin: 0px;
}
</style>
