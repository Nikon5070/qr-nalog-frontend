<template>
  <div class="qr-scanner">
    <div class="container">
      <video ref="video"></video>
      <canvas ref="canvas" :height="canvasHeight"
              :width="canvasWidth"></canvas>
    </div>
  </div>
</template>

<script lang="ts">
import {ref, defineComponent} from 'vue'

import jsQR from "jsqr";

export default defineComponent({
  name: 'QrScanner',
  setup: () => {
    const count = ref(0)
    const canvasHeight = ref(0)
    const canvasWidth = ref(0)
    const canvas2d = ref<null | CanvasRenderingContext2D>(null)

    return {
      count, canvasHeight,
      canvasWidth, canvas2d,
    }
  },
  mounted() {
    navigator.mediaDevices.getUserMedia({video: true, audio: false})
        .then((stream) => {
          this.$refs.video.srcObject = stream;
          this.$refs.video.play();
          requestAnimationFrame(this.tick);
        })
        .catch(function (err) {
          console.log("An error occurred: " + err);
        });
  },
  methods: {
    tick: function () {
      const video = this.$refs.video
      const canvas = this.$refs.canvas
      if (video.readyState === video.HAVE_ENOUGH_DATA) {
        this.canvasHeight = video.videoHeight;
        this.canvasWidth = video.videoWidth;
        !this.canvas2d &&
        (this.canvas2d = canvas.getContext("2d"));

        this.canvas2d?.drawImage(
            video,
            0,
            0,
            this.canvasWidth,
            this.canvasHeight,
        );
        //
        // {
        //   "t": "20210811T1655",
        //     "s": "275.48",
        //     "fn": "9960440300126733",
        //     "i": "45819",
        //     "fp": "2774627351",
        //     "n": "1"
        // }
        const imageData = this.canvas2d?.getImageData(
            0,
            0,
            this.canvasWidth,
            this.canvasHeight
        );
        const code = jsQR(imageData.data, imageData.width, imageData.height, {
          inversionAttempts: "dontInvert"
        });

        if (code) {
          console.log(this.parseQr(code.data))
        }
      }
      requestAnimationFrame(this.tick);
    },
    parseQr(value: string) {
      if (!value) return null

      return value.split('&').reduce((acc, cur) => {
        const [key, v] = cur.split('=')
        return {...acc, [key]: v}
      }, {})
    }
  }
})
</script>

<style scoped>
.qr-scanner {
  display: flex;
  flex-direction: column;
  max-width: 600px;
  width: 100%;
}

video {
  visibility: hidden;
  opacity: 0;
  pointer-events: none;
  width: 0;
  height: 0;
  position: fixed;
  top: -100vh;
  right: -100vw;
}

canvas {
  display: block;
}
</style>
