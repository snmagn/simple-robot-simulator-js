<template>
  <div>
    <div v-for="(line, index) in mapData" :key="index">
      <div class="one-line" v-for="tile in mapData[index]" :key="tile">
        <MapTile v-bind:type="tile"></MapTile>
      </div>
    </div>
    <!-- <MapTile></MapTile> -->
  </div>
</template>

<script>
import MapTile from './MapTile.vue'
export default {
  components: { MapTile },
  props: {
    mapData: {
      type: Array,
      required: false,
      default: [
        [1,1,1,1,1,1,1],
        [1,1,1,1,1,1,1],
        [1,1,1,1,1,1,1],
      ],
    },
  },
}
</script>

<style>
.one-line {
  display: inline-block;
}
</style>