<template>
  <div>
    <div id="app">Hello world</div>
    <input
      @change="increaseVolume"
      class="gain-control"
      type="range"
      min="0"
      max="1"
      step="0.01"
      value="0.5"
    />
  </div>
</template>

<script>
export default {
  name: "app",
  data() {
    return {
      context: null,
      stream: null,
      lineInSource: null,
      gainNode: null
    };
  },
  async mounted() {
    this.context = new AudioContext();
    // if (context.state === "suspended") {
    //   await context.resume();
    // }
    this.stream = await navigator.mediaDevices.getUserMedia({
      audio: {
        echoCancellation: false,
        autoGainControl: false,
        noiseSuppression: false,
        latency: 0
      }
    });
    this.lineInSource = this.context.createMediaStreamSource(this.stream);
    this.lineInSource.connect(this.context.destination);
    this.gainNode = new GainNode(this.context, { gain: 0.5 });
    this.lineInSource.connect(this.gainNode).connect(this.context.destination);
  },
  methods: {
    increaseVolume(event) {
      const parsed = parseFloat(event.target.value);
      const value = Number.isNaN(parsed) ? 1 : parsed;
      const clamped = this.clamp(0, 1, value);
      console.log(clamped);

      this.gainNode.gain.setTargetAtTime(
        clamped,
        this.context.currentTime,
        0.01
      );
      console.log(this.gainNode);
    },
    clamp(min, max, value) {
      console.log(Math.min(Math.max(value, min), max));
      return Math.min(Math.max(value, min), max);
    }
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
