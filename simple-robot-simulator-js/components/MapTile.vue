<template>
  <div class="square" v-bind:style="bgColor">
  </div>
</template>

<script>
export default {
  props: {
    type: {
      type: String,
      required: false,
      default: "Wall",
    }
  },
  computed: {
    // 算出 getter 関数
    bgColor: function () {
      // `this` は vm インスタンスを指します
      var bgColors = {
        Wall: "rgb(44, 44, 44)",
        Floor: "#eee",
      }
      return {
        '--back-ground': bgColors[this.type],
      }
    }
  }
}
</script>

<style>
.square{
  position: relative;
  width: 100%;
  padding-top: 100%;
  background: var(--back-ground)
}
</style>