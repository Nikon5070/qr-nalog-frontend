<template>
  <div class="qr-scanner">
    <video ref="video"></video>
    <canvas ref="canvas"></canvas>
  </div>
</template>

<script lang="ts">
import {ref, defineComponent} from 'vue'

import jsQR from "jsqr";

export default defineComponent({
  name: 'QrScanner',
  setup: () => {
    const count = ref(0)
    return {count}
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
    tick() {
      console.log('Alo')
    }
  }
})
</script>

<style scoped>
code {
  background-color: #eee;
  padding: 2px 4px;
  border-radius: 4px;
  color: #304455;
}
</style>
