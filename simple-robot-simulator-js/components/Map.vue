<template>
  <table>
    <tr v-for="(line, index) in mapData" :key="index">
      <MapTile v-for="(type, mIndex) in mapData[index]" :key="mIndex" v-bind:type="type" v-bind:loc="{x:mIndex, y:index}" v-bind:rot="robot.rot" v-bind:robotLoc="robot.loc"></MapTile>
    </tr>
    <!-- <MapTile></MapTile> -->
  </table>
</template>

<script>
import MapTile from './MapTile.vue'
export default {
  components: { MapTile },
  props: {
    robot: {
      type: Object,
      required: false,
      default: () => ({
        loc: {
          x:0,
          y:0,
        },
        rot: 0,
      })
    },
    mapData: {
      type: Array,
      required: true,
    },
  },
  created: function () {
    this.mapData.forEach((line, y) => {
      line.forEach((tile, x) => {
        if (tile == 0) {
          this.robot.loc.x = x
          this.robot.loc.y = y
        }
      });
    });
  }
}
</script>

<style>
.one-line {
  display: inline-block;
}
</style>