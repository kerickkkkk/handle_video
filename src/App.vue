<script setup>
import { ref} from 'vue'
const video= ref(null)
 const startCam = () => {
  if (video.value.srcObject) {
    alert('鏡頭已打開');
    return;
  }
  const constraints = { audio: false, video: true }
  navigator.mediaDevices.getUserMedia(constraints)
  .then((stream) => {
    video.value.srcObject = stream;
    video.value.onloadedmetadata = () => {
      video.value.play();
    };
  })
  .catch((err) => {
    alert(err.message);
  });
 }
 const stopCam = () => {
  if (!video.value.srcObject) {
    alert('請打開鏡頭');
    return;
  }
  const stream = video.value.srcObject;
  const tracks = stream.getTracks();

  tracks.forEach((track) => {
    track.stop();
  });
  video.value.srcObject = null;

}
const capturePic = () => {
  if (!video.value.srcObject) {
    alert('請打開鏡頭');
    return;
  }
  const canvas = document.createElement('canvas');
  canvas.width = video.value.videoWidth;
  canvas.height = video.value.videoHeight;
  const ctx = canvas.getContext('2d');
  ctx.drawImage(video.value, 0, 0, canvas.width, canvas.height);

  // 下載截圖
  const link = document.createElement('a');
  link.href = canvas.toDataURL('image/png');
  link.download = `截圖-${new Date().getTime()}.png`;
  link.click();
}


</script>

<template>
<video ref="video" width="500"></video>
<hr>
<button type="button" @click="startCam">打開視訊</button>
<button type="button" @click="stopCam">停止視訊</button>
<button type="button" @click="capturePic">截圖</button>
</template>

<style scoped>

</style>
