<template>
  <div class="square" v-bind:style="bgColor">
  </div>
</template>

<script>
export default {
  props: {
    type: {
      type: Number,
      required: false,
      default: 1,
    }
  },
  computed: {
    // 算出 getter 関数
    bgColor: function () {
      // `this` は vm インスタンスを指します
      var bgColors = {
        1: "rgb(44, 44, 44)",
        2: "#eee",
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
  display: inline-block;
  width: 40px;
  height: 40px;
  background: var(--back-ground)
}
</style>