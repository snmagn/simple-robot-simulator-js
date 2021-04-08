<template>
  <div class="container">
    <div>
      <Map v-bind:robot="robot"></Map>
      <b-button v-on:click="run" variant="success">Run</b-button>
      <b-button v-on:click="step" variant="success">Step</b-button>
      <b-button v-on:click="stop" variant="danger">Stop</b-button>
    </div>
  </div>
</template>

<script>
import Map from '../components/Map.vue'
export default {
  components: { Map },
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
    intervalTimerHandler: null,
    programChain: [],
    programCounter: 0,
    prevResult: 0,
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
    processRun: function () {
      var action = this.programChain[this.programCounter++]
      if (this.programCounter >= this.programChain.length) {
        this.programCounter = 0
      }
      switch (action.action) {
        case 'sensor':
          this.prevResult = 0
          if (this.robot.rot < 90) {
            var sensor = this.robot.loc.y -1
            if (sensor > 0) {
              if (this.mapData[sensor][this.robot.loc.x]) {
                this.prevResult = 1
              }
            } else {
              this.prevResult = 1
            }
          } else if (this.robot.rot < 180) {
            var sensor = this.robot.loc.x +1
            if (sensor < this.mapData[0].length) {
              if (this.mapData[this.robot.loc.y][sensor]) {
                this.prevResult = 1
              }
            } else {
              this.prevResult = 1
            }
          } else if (this.robot.rot < 270) {
            var sensor = this.robot.loc.y +1
            if (sensor < this.mapData.length) {
              if (this.mapData[sensor][this.robot.loc.x]) {
                this.prevResult = 1
              }
            } else {
              this.prevResult = 1
            }
          } else if (this.robot.rot < 360) {
            var sensor = this.robot.loc.x -1
            if (sensor > 0) {
              if (this.mapData[this.robot.loc.y][sensor]) {
                this.prevResult = 1
              }
            } else {
              this.prevResult = 1
            }
          }
          break;
        case 'go':
          var dv = this.calcDirection()
          this.robot.loc.x += dv.x
          this.robot.loc.y += dv.y
          if (this.robot.loc.x <= 0) {
            this.robot.loc.x = 0
          } else if (this.robot.loc.x >= this.mapData[0].length) {
            this.robot.loc.x = this.mapData[0].length - 1
          }
          if (this.robot.loc.y <= 0) {
            this.robot.loc.y = 0
          } else if (this.robot.loc.y >= this.mapData.length) {
            this.robot.loc.y = this.mapData.length - 1
          }
          this.prevResult = 1
          break;
        case 'rot':
          this.robot.rot += action.params[0]
          this.prevResult = 1
          break;
      
        default:
          break;
      }
    },
    program: function () {
      // ここにプログラムを書く
      this.actionGoStraight()
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
