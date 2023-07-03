<script setup>
import { ref, computed } from 'vue'
const video= ref(null)
const deviceList = ref([])
const currentDeviceId = ref('')

const enumerateDevices = () => {
  return new Promise((resolve, reject) => {
    navigator.mediaDevices.enumerateDevices()
      .then(devices => {
        const videoDevices = devices.filter(device => device.kind === 'videoinput');
        const audioDevices = devices.filter(device => device.kind === 'audioinput');

        const videoDeviceOptions = videoDevices.map(device => ({ label: device.label, deviceId: device.deviceId }));
        const audioDeviceOptions = audioDevices.map(device => ({ label: device.label, deviceId: device.deviceId }));

        const deviceInfo = {
          videoDevices: videoDeviceOptions,
          audioDevices: audioDeviceOptions
        };
        deviceList.value = deviceInfo.videoDevices
        resolve(deviceInfo);
      })
      .catch(err => {
        reject(err);
      });
  });
};
const handleCam = async() => {
  try {
    if (video.value.srcObject) {
      alert('鏡頭已打開');
      return;
    }
    let constraints = null
    if( currentDeviceId.value ){
      constraints = { audio: false, video : {
        deviceId : currentDeviceId.value
      } }
    } else {
      // 沒有先啟用 會抓不到裝置
      constraints = { audio: false, video : true }
    }

    const stream = await navigator.mediaDevices.getUserMedia(constraints)
    video.value.srcObject = stream;
    video.value.onloadedmetadata = () => {
      video.value.play();
    };
    await enumerateDevices() 
  } catch (error) {
    alert(error.message);
  }
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

const cuurentLabel = computed(()=>{
  return currentDeviceId.value === '' ? '' :` : ${(deviceList.value.find(item => item.deviceId === currentDeviceId.value)).label}`
})

</script>

<template>
<video ref="video" width="500"></video>
<hr>

<button type="button" @click="handleCam">
  {{`開啟視訊裝置 ${ cuurentLabel }`}}
</button>
<button type="button" @click="stopCam">停止視訊</button>
<button type="button" @click="capturePic">截圖</button>

<br />
<br />
  <template v-if="deviceList.length > 0">
    <label for="deviceList">選擇視訊裝置：
    </label>
    <select 
      id="deviceList"
      v-model="currentDeviceId"
      @change = 'handleCam'>
      <option 
        v-for="device in deviceList"
        :key="device.deviceId"
        :value="device.deviceId" >{{ 
          device.label
        }}</option>
    </select>
  </template>

</template>

<style scoped>

</style>
