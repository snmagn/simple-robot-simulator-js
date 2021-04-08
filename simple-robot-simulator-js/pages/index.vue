<template>
  <div class="container">
    <div>
      <Map v-bind:robot="robot"></Map>
      <b-button v-on:click="run" variant="success">Run</b-button>
      <b-button v-on:click="step" variant="success">Step</b-button>
      <b-button v-on:click="stop" variant="danger">Stop</b-button>
      <div>{{ steps }} steps</div>
      <div>{{ msg }}</div>
    </div>
  </div>
</template>

<script>
import Map from '../components/Map.vue'
export default {
  components: { Map },
  props: {
    msg: {
      type: String,
      required: false,
      default: () => "",
    },
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
      required: false,
      default: () => [
        [1, 1, 1, 1, 1, 1, 1, 1, 1],
        [1, 9, 2, 2, 2, 2, 2, 2, 1],
        [1, 1, 1, 1, 1, 1, 1, 2, 1],
        [1, 1, 1, 1, 1, 1, 1, 0, 1],
        [1, 1, 1, 1, 1, 1, 1, 1, 1],
      ],
    },
  },
  data: () => ({
    steps: 0,
    intervalTimerHandler: null,
    programChain: [],
    programCounter: 0,
    prevResult: 0,
    skipNext: false,
    vars: [],
  }),
  created: function () {
    this.program()
  },
  methods: {
    run: function () {
      this.intervalTimerHandler = setInterval(() => {
        this.processRun()
      }, 1000);
    },
    step: function () {
      this.stop()
      this.processRun()
    },
    stop: function () {
      clearInterval(this.intervalTimerHandler)
    },
    calcDirection: function () {
      var dv = {
        x: 0,
        y: 0,
      }
      if (this.robot.rot < 90) {
        dv.y -= 1
      } else if (this.robot.rot < 180) {
        dv.x += 1
      } else if (this.robot.rot < 270) {
        dv.y += 1
      } else if (this.robot.rot < 360) {
        dv.x -= 1
      }
      return dv
    },
    actionAdd: function (action, ...params) {
      this.programChain.push({
        action: action,
        params: params,
      })
    },
    actionSensor: function () {
      this.actionAdd('sensor')
    },
    actionGoStraight: function () {
      this.actionAdd('go')
    },
    actionRotate: function (degree) {
      this.actionAdd('rot', degree)
    },
    actionMemory: function (varName) {
      this.actionAdd('memory', varName)
    },
    actionCondition: function (...condition) {
      this.actionAdd('condition', condition)
    },
    processSensor: function () {
      var result = 0

      if (this.robot.rot < 90) {
        var sensor = this.robot.loc.y -1
        if (sensor > 0) {
          if (this.mapData[sensor][this.robot.loc.x]) {
            result = 1
          }
        } else {
          result = 1
        }
      } else if (this.robot.rot < 180) {
        var sensor = this.robot.loc.x +1
        if (sensor < this.mapData[0].length) {
          if (this.mapData[this.robot.loc.y][sensor]) {
            result = 1
          }
        } else {
          result = 1
        }
      } else if (this.robot.rot < 270) {
        var sensor = this.robot.loc.y +1
        if (sensor < this.mapData.length) {
          if (this.mapData[sensor][this.robot.loc.x]) {
            result = 1
          }
        } else {
          result = 1
        }
      } else if (this.robot.rot < 360) {
        var sensor = this.robot.loc.x -1
        if (sensor > 0) {
          if (this.mapData[this.robot.loc.y][sensor]) {
            result = 1
          }
        } else {
          result = 1
        }
      }
      
      return result
    },
    processGo: function () {
      var result = 0
      var dv = this.calcDirection()
      var nextX = this.robot.loc.x + dv.x
      var nextY = this.robot.loc.y + dv.y
      if (nextX <= 0) {
        nextX = 0
      } else if (nextX >= this.mapData[0].length) {
        nextX = this.mapData[0].length - 1
      }
      if (nextY <= 0) {
        nextY = 0
      } else if (nextY >= this.mapData.length) {
        nextY = this.mapData.length - 1
      }

      // 壁にめり込んでいるか
      if (this.mapData[nextY][nextX] == 1) {
        // 壁にめり込んでいる場合は位置更新を行わない
        result = 1
      } else {
        // 壁にめり込んでいない場合は位置更新を行う
        this.robot.loc.x = nextX
        this.robot.loc.y = nextY
      }

      // ゴールしているか？
      if (this.mapData[this.robot.loc.y][this.robot.loc.x] == 9) {
        // 実行を停止し、完了メッセージを表示する。
        this.stop()
        this.msg = "ゴール！！！"
      }

      return result
    },
    processRotate: function (degree) {
      var result = 0
      this.robot.rot += degree
      if (this.robot.rot <= 0 || this.robot.rot >= 360) {
        this.robot.rot = 0
      }
      return result
    },
    processMemory: function (varName) {
      var result = 0
      this.vars[varName] = this.prevResult
      return result
    },
    processCondition: function (...condition) {
      if (condition.length == 1) {
        if (this.prevResult != condition[0]) {
          this.skipNext = true
        }
      } else if(this.vars[condition[1]] != condition[0]) {
          this.skipNext = true
      }
    },
    processRun: function () {
      this.steps++
      var action = this.programChain[this.programCounter++]
      if (this.programCounter >= this.programChain.length) {
        this.programCounter = 0
      }
      if (this.skipNext) {
        this.skipNext = false
        return
      }
      switch (action.action) {
        case 'sensor':
          this.prevResult = this.processSensor()
          break;
        case 'go':
          this.prevResult = this.processGo()
          break;
        case 'rot':
          this.prevResult = this.processRotate(action.params[0])
          break;
        case 'memory':
          this.prevResult = this.processMemory(action.params[0])
          break;
        case 'condition':
          this.prevResult = this.processCondition(action.params)
          break;
      
        default:
          break;
      }
    },
    program: function () {
      // ここにプログラムを書く
      this.actionGoStraight()
      this.actionSensor()
      this.actionCondition(1)
      this.actionRotate()
    }
  },
}
</script>

<style>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.title {
  font-family:
    'Quicksand',
    'Source Sans Pro',
    -apple-system,
    BlinkMacSystemFont,
    'Segoe UI',
    Roboto,
    'Helvetica Neue',
    Arial,
    sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}
</style>
