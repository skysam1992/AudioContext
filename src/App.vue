<template>
  <div class="container">
    <canvas class="canvas" ref="canvas" width="1080" height="200"></canvas>
  </div>
  <div class="btn" @click="handleClick"></div>
</template>

<script>
export default {
  data() {
    return {
      audioContext: undefined,
      canvasContext: undefined,
      analyser: undefined,
      dataArray: [],
      array:[0, 1, 2],
      bufferLength: 0,
      obj: { f1: 'foo', f2: 'bar' }
    }
  },
  mounted() {
    console.log(this.obj)

    console.dir(this.obj)

    this.initAudioContext()
    this.initCanvas()
  },
  methods: {
    handleClick() {
      this.playAudio()
    },
    async initAudioContext() {
      this.audioContext = new AudioContext()
    },
    async playAudio() {
      const res = await fetch('http://aimg8.dlszyht.net.cn/ueditor/file/823/1644869/1553422829215244.mp3')
      const arrayBuffer = await res.arrayBuffer()
      const audioBuffer = await this.audioContext.decodeAudioData(arrayBuffer)
      const source = this.audioContext.createBufferSource()
      source.buffer = audioBuffer
      
      this.analyser = this.audioContext.createAnalyser()
      this.analyser.fftSize = 512

      source.connect(this.analyser)
      this.analyser.connect(this.audioContext.destination)
      
      source.start()

      this.bufferLength = this.analyser.frequencyBinCount
      this.dataArray = new Uint8Array(this.bufferLength)

      this.draw()
    },
    initCanvas() {
      this.canvasContext = this.$refs.canvas.getContext('2d')
    },
    draw() {
      requestAnimationFrame(this.draw)
      const width = this.$refs.canvas.width
      const height = this.$refs.canvas.height
      const barWidth = width / this.bufferLength * 1.5
      let barHeight = 0
      this.analyser.getByteFrequencyData(this.dataArray)
      this.canvasContext.clearRect(0, 0, width, height)
      for (var i = 0, x = 0; i < this.bufferLength; i++) {
        barHeight = this.dataArray[i]
        console.log('barHeight', barHeight);
        var r = barHeight + 25 * (i / this.bufferLength)
        var g = 250 * (i / this.bufferLength)
        var b = 50

        this.canvasContext.fillStyle = "rgb(" + r + "," + g + "," + b + ")"
        this.canvasContext.fillRect(x, height - barHeight, barWidth, barHeight)

        x += barWidth + 2
      }
    }
  }
}
</script>

<style scoped>
.container {
  display: flex;
  align-items: center;
}
.canvas {
  width: 1080px;
  height: 200px;
}
.btn {
  width: 80px;
  height: 40px;
  line-height: 40px;
  border: 1px solid red;
}
</style>
