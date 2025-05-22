<template>
  <div class="camera-snapshot">
    <button v-if="!videoStarted" @click="startVideo">📸 Включить камеру</button>

    <div v-if="isCameraSupported && videoStarted" class="camera-container">
      <video ref="video" autoplay playsinline></video>
      <button @click="takePhoto">📸 Сделать снимок</button>
    </div>

    <div v-if="!isCameraSupported" class="fallback-container">
      <p>Камера не поддерживается или не разрешена.</p>
    </div>
    
    <canvas ref="canvas" style="display: none;"></canvas>
    <img v-if="photoDataUrl" :src="photoDataUrl" alt="Сделанное фото" />
  </div>
</template>

<script>
export default {
    data() {
        return {
            videoStarted: false,
            photoDataUrl: null,
            stream: null,
        }
    },

    computed: {
        isCameraSupported() {
            return !!(navigator.mediaDevices && navigator.mediaDevices.getUserMedia);
        }
    },

    methods: {
      async startVideo() {
        try {
          this.videoStarted = true;
            this.stream = await navigator.mediaDevices.getUserMedia({
                video: { facingMode: 'environment' },
                audio: false
            });
            this.$refs.video.srcObject = this.stream;
        } catch (e) {
          this.videoStarted = false;
            console.error(e);
        }
      },

      takePhoto() {
        const video = this.$refs.video;
        const canvas = this.$refs.canvas;

        if (!video || video.videoWidth === 0) {
          alert('Камера еще не готова.');
          return;
        }

        canvas.width = video.videoWidth;
        canvas.height = video.videoHeight;
        const ctx = canvas.getContext('2d');
        ctx.drawImage(video, 0, 0, canvas.width, canvas.height);

        this.photoDataUrl = canvas.toDataURL('image/png');
      }
    }
}
</script>

<style scoped>
.camera-snapshot {
  display: flex;
  flex-direction: column;
  gap: 8px;
  justify-content: center;
  align-items: center;
}
.camera-container {
  display: flex;
  flex-direction: column;
  align-items: center;
}
video, img {
  width: 100%;
  max-width: 400px;
  margin-bottom: 10px;
}
</style>
