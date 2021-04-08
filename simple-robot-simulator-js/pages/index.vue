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
  data: {
    intervalTimerHandler: null,
  },
  methods: {
    run: function () {
      this.intervalTimerHandler = setInterval(() => {
        this.robot.loc.y -= 1
      }, 1000);
    },
    step: function () {
      this.intervalTimerHandler = setInterval(() => {
        this.robot.loc.y -= 1
        clearInterval(this.intervalTimerHandler)
      }, 0);
    },
    stop: function () {
      clearInterval(this.intervalTimerHandler)
    }
  }
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
