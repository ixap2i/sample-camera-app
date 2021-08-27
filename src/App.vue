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
