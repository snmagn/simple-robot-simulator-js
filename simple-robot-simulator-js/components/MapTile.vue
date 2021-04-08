<template>
  <div class="square" v-bind:style="bgColor">
    <img class="robot" v-show="isHere" v-bind:style="robotRot" src="~/assets/robot.png"/>
  </div>
</template>

<script>
export default {
  props: {
    type: {
      type: Number,
      required: false,
      default: 1,
    },
    /// ロボットがこのタイルにいる。
    isHere: {
      type: Boolean,
      required: false,
      default: false,
    },
    /// 0:上向き,時計回り（〜359）,現在は90刻みでしか動作しない。
    rot: {
      type: Number,
      required: false,
      default: 0,
    }
  },
  computed: {
    // 算出 getter 関数
    bgColor: function () {
      // `this` は vm インスタンスを指します
      var bgColors = {
        // スタート
        0: "rgb(44, 200, 200)",
        // 壁
        1: "rgb(44, 44, 44)",
        // 床
        2: "#eee",
        // ゴール
        9: "rgb(200, 44, 44)",
      }
      return {
        '--back-ground': bgColors[this.type],
      }
    },
    robotRot: function () {
      return {
        '--robot-rot': this.rot + "deg",
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
.robot {
  height: 100%;
  width: 100%;
  transform: rotate(var(--robot-rot));
}
</style>