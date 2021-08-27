<template>
  <img alt="Vue logo" src="./assets/logo.png">
  <HelloWorld msg="Welcome to Your Vue.js App"/>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
import Quagga from 'quagga';

export default {
  name: 'App',
  components: {
    HelloWorld
  },
  mounted: function() {
    this.startCamera()
    this.loadCamera()
  },
  methods: {
    startCamera: function() {
      Quagga.init({
        inputStream: {
          name: 'Live',
          type: 'LiveStream',
          target: document.querySelector('#cameraDemo')
        },
        decoder: {
          readers: ['ean_reader']
        }
      },
      function(err) {
        if (err) {
          console.log(err);
          return
        }
        console.log('Initialize was completed.');
        Quagga.start();
      })
      Quagga.onDetected(function(result) {
        console.log(result.codeResult);
        if (result.codeResult) {
          console.log(result.codeResult.code);
        }
      });
    },
    loadCamera: function() {
      const camera = document.querySelector('#cameraDemo');
      const canvas = document.querySelector('#picture');
      document.querySelector('#shutter').addEventListener('click', function(){
        const ctx = canvas.getContext('2d');

        camera.pause();
        setTimeout(() => {
          camera.play();
        }, 500)

        ctx.drawImage(camera, 0, 0, canvas.width, canvas.height);
      });

      const constrains = {
        audio: false,
        video: {
          width: 200,
          height: 350,
          facingMode: 'user'
        // facingMode: { exact: "environment" }  // リアカメラを利用する場合
        }
      }
      navigator.mediaDevices.getUserMedia(constrains)
        .then(((stream)=>{
          camera.srcObject = stream;
          onloadedmetadata = () => { camera.play };
        })).catch(((err)=>{ console.log(err) }))
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
