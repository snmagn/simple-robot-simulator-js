<template>
  <div class="container">
    <div>
      <Map ref="map" v-bind:robot="robot" v-bind:mapData="mapDataList[selected]"></Map>
      <b-button v-on:click="run" variant="success">Run</b-button>
      <b-button v-on:click="step" variant="success">Step</b-button>
      <b-button v-on:click="stop" variant="danger">Stop</b-button>
      <b-button v-on:click="reset" variant="danger">Reset</b-button>
      <div>{{ steps }} steps</div>
      <div>{{ msg }}</div>
    </div>
    <div>
      <select v-model="selected">
        <option v-for="(mapData, name) in mapDataList" :key="name">{{ name }}</option>
      </select>
      <div>{{ intervalTime }} ms</div>
      <vue-slider
        ref="slider"
        v-model="intervalTime"
      ></vue-slider>
    </div>
  </div>
</template>

<script>
import VueSlider from 'vue-slider-component'
import 'vue-slider-component/theme/default.css'
import Map from '../components/Map.vue'
export default {
  components: { Map, VueSlider },
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
    mapDataList: {
      type: Object,
      required: false,
      default: () => ({
        level1: [
          [1, 1, 1, 1, 1, 1, 1, 1, 1],
          [1, 9, 2, 2, 2, 2, 2, 2, 1],
          [1, 1, 1, 1, 1, 1, 1, 2, 1],
          [1, 1, 1, 1, 1, 1, 1, 0, 1],
          [1, 1, 1, 1, 1, 1, 1, 1, 1],
        ],
        level2: [
          [1, 1, 1, 1, 1, 1, 1, 1, 1],
          [1, 9, 2, 2, 2, 2, 2, 1, 1],
          [1, 1, 1, 1, 1, 1, 2, 2, 1],
          [1, 1, 1, 1, 1, 1, 1, 0, 1],
          [1, 1, 1, 1, 1, 1, 1, 1, 1],
        ],
        level3: [
          [1, 1, 1, 1, 1, 1, 1, 1, 1],
          [1, 9, 1, 2, 2, 2, 2, 2, 1],
          [1, 2, 1, 2, 1, 2, 1, 2, 1],
          [1, 2, 1, 2, 1, 2, 1, 2, 1],
          [1, 2, 2, 2, 2, 2, 1, 0, 1],
          [1, 1, 1, 1, 1, 1, 1, 1, 1],
        ],
        level4: [
          [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
          [1, 9, 1, 2, 2, 2, 2, 2, 2, 2, 1, 2, 1],
          [1, 2, 1, 1, 1, 2, 1, 2, 1, 2, 2, 2, 1],
          [1, 2, 2, 2, 1, 2, 1, 2, 1, 1, 1, 2, 1],
          [1, 1, 1, 2, 1, 2, 1, 0, 1, 2, 2, 2, 1],
          [1, 1, 1, 2, 2, 2, 1, 1, 1, 2, 1, 2, 1],
          [1, 1, 1, 1, 1, 2, 1, 9, 1, 2, 2, 1, 1],
          [1, 1, 1, 1, 9, 2, 1, 2, 1, 1, 2, 1, 1],
          [1, 1, 1, 1, 1, 1, 1, 2, 2, 2, 2, 1, 1],
          [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
        ],
      })
    },
  },
  data: () => ({
    intervalTimerHandler: null,
    intervalTime: 50,
    programChain: [],
    selected: "level1",
    msg: "",
    steps: 0,
    programCounter: 0,
    prevResult: 0,
    skipNext: false,
    vars: [],
  }),
  created: function () {
    this.program()
  },
  methods: {
    typeOf: function (obj) {
      return Object.prototype.toString.call(obj).slice(8, -1).toLowerCase();
    },
    run: function () {
      this.intervalTimerHandler = setInterval(() => {
        this.processRun()
      }, this.intervalTime);
    },
    step: function () {
      this.stop()
      this.processRun()
    },
    stop: function () {
      clearInterval(this.intervalTimerHandler)
    },
    reset: function () {
      this.stop()
      this.msg = ""
      this.steps = 0
      this.programCounter= 0
      this.prevResult= 0
      this.skipNext = false
      this.vars = []
      // this.$refs.map.$emit('reset')
      this.$refs.map.reset()
      console.debug("reset")
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
    actionMemory: function (varName, value = null) {
      this.actionAdd('memory', varName, value)
    },
    actionCondition: function (condition, varName, registerable = false) {
      this.actionAdd('condition', condition, varName, registerable)
    },
    processSensor: function () {
      console.log("process Sensor")
      var result = false

      var nextLoc = this.calcNextLoc()

      if (this.mapDataList[this.selected][nextLoc.y][nextLoc.x] == 1) {
        result = true
      }
      console.log("sensor x:" + nextLoc.x + ", y:" + nextLoc.y)
      console.log("sensor:" + result)
      
      return result
    },
    calcNextLoc: function () {
      var dv = this.calcDirection()
      var nextX = this.robot.loc.x + dv.x
      var nextY = this.robot.loc.y + dv.y
      if (nextX <= 0) {
        nextX = 0
      } else if (nextX >= this.mapDataList[this.selected][0].length) {
        nextX = this.mapDataList[this.selected][0].length - 1
      }
      if (nextY <= 0) {
        nextY = 0
      } else if (nextY >= this.mapDataList[this.selected].length) {
        nextY = this.mapDataList[this.selected].length - 1
      }

      console.log("nextLoc: x="+nextX+", y="+nextY)

      return {
        x: nextX,
        y: nextY,
      }
    },
    processGo: function () {
      console.log('process Go')
      var result = false
      var nextLoc = this.calcNextLoc()
      var nextX = nextLoc.x
      var nextY = nextLoc.y

      // 壁にめり込んでいるか
      if (this.mapDataList[this.selected][nextY][nextX] == 1) {
        // 壁にめり込んでいる場合は位置更新を行わない
        result = true
      } else {
        // 壁にめり込んでいない場合は位置更新を行う
        this.robot.loc.x = nextX
        this.robot.loc.y = nextY
      }

      // ゴールしているか？
      if (this.mapDataList[this.selected][this.robot.loc.y][this.robot.loc.x] == 9) {
        // 実行を停止し、完了メッセージを表示する。
        this.stop()
        this.msg = "ゴール！！！"
      }

      return result
    },
    processRotate: function (degree) {
      console.log("process Rotate")
      var result = false
      this.robot.rot += degree
      if (this.robot.rot <= 0) {
        this.robot.rot = 0
      } else if (this.robot.rot >= 360) {
        this.robot.rot -= 360
      }
      return result
    },
    processMemory: function (varName, value = null) {
      console.log("process Memory")
      var result = false
      if (value == null) {
        value = this.prevResult
      }
      this.vars[varName] = value
      console.log("memory[" + varName + "]:" + this.vars[varName])
      return result
    },
    conditionEqual: function (condition, varName) {
      var result = true
      console.log("condition Equal")
      console.log("condition:" + condition + ", varName:" + varName)
      if (varName == null) {
        console.log("condtion prevResult:" + this.prevResult)
        if (this.prevResult != condition) {
          result = false
        }
      } else if(this.typeOf(varName) == "string") {
        console.log("condtion var:" + this.vars[varName])
        var value = this.vars[varName]
        if (value === void 0) {
          value = false
        }
        if(value != condition) {
          result = false
        }
      } else if(this.typeOf(varName) == "function") {
        if((varName() != condition)) {
          result = false
        }
        console.log("condtion function:" + result)
      } else {
        console.error("invalid condition", varName)
      }
      console.log("condition Equal[result]:" + result)
      return result
    },
    processCondition: function (condition, varName) {
      var result = true
      console.log("process Condition")
      console.log("condition:" + condition + ", varName:" + varName)
      if (varName == null) {
        console.log("condtion prevResult:" + this.prevResult)
        if (this.prevResult != condition) {
          this.skipNext = true
          result = false
        }
      } else if(this.typeOf(varName) == "string") {
        console.log("condtion var:" + this.vars[varName])
        var value = this.vars[varName]
        if (value === void 0) {
          value = false
        }
        if(value != condition) {
          this.skipNext = true
          result = false
        }
      } else if(this.typeOf(varName) == "function") {
        if((varName() != condition)) {
          this.skipNext = true
          result = false
        }
        console.log("condtion function:" + result)
      } else {
        console.error("invalid condition", varName)
      }
      console.log("condition skip:" + this.skipNext)
      return result
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
          if (action.params.length == 1) {
            this.prevResult = this.processMemory(action.params[0])
          } else {
            this.prevResult = this.processMemory(action.params[0], action.params[1])
          }
          break;
        case 'condition':
          if (action.params.length == 1) {
            this.prevResult = this.processCondition(action.params[0], null)
          } else {
            this.prevResult = this.processCondition(action.params[0], action.params[1])
          }
          break;
      
        default:
          break;
      }
    },
    program: function () {
      // ここにプログラムを書く
      this.actionGoStraight()
      this.actionSensor()
      this.actionCondition(true)
      this.actionRotate(90)
      this.actionSensor()
      this.actionCondition(true)
      this.actionRotate(180)
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
