/* eslint-disable vue/no-mutating-props */
/* eslint-disable vue/no-mutating-props */
<template>
  <img alt="Vue logo" src="./assets/logo.png">
  {{this.barcodeTxt}}
  {{this.event}}
  <HelloWorld v-on:event="this.barcode = this.barcodeTxt" />
  <!-- <HelloWorld v-bind:barcode="this.barcodeTxt" /> -->
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
import Quagga from 'quagga';

export default {
  name: 'App',
  components: {
    HelloWorld
  },
  data: function() {
    return {
      barcodeTxt: 'test',
    };
  },
  computed: {
    targetArea: function() {
      return document.querySelector('#barcodeField').innerHTML;
    }
  },
  props: {
    barcodeEvent: {
      type: Function
    }
  },
  watch: {
    barcodeWatcher: function(eve) {
      this.barcodeTxt = eve;
      console.log(eve);
    }
  },
  mounted: function() {
    const camera = document.querySelector('#cameraDemo');

    this.startCamera()
    this.loadCamera(camera)
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


      Quagga.onProcessed(function(result) {
        var drawingCtx = Quagga.canvas.ctx.overlay,
        drawingCanvas = Quagga.canvas.dom.overlay;

        if (result) {
          if (result.boxes) {
              drawingCtx.clearRect(0, 0, parseInt(drawingCanvas.getAttribute("width")), parseInt(drawingCanvas.getAttribute("height")));
              result.boxes.filter(function (box) {
                  return box !== result.box;
              }).forEach(function (box) {
                  Quagga.ImageDebug.drawPath(box, {x: 0, y: 1}, drawingCtx, {color: "green", lineWidth: 2});
              });
          }

          if (result.box) {
              Quagga.ImageDebug.drawPath(result.box, {x: 0, y: 1}, drawingCtx, {color: "#00F", lineWidth: 2});
          }

          if (result.codeResult && result.codeResult.code) {
              Quagga.ImageDebug.drawPath(result.line, {x: 'x', y: 'y'}, drawingCtx, {color: 'red', lineWidth: 3});
          }
        }
      });

      Quagga.onDetected(function(result) {
        console.log(result.codeResult);
        if (result.codeResult) {
          console.log(result.codeResult.code);
          var test = document.querySelector('#barcodeField');
          this.barcodeTxt = result.codeResult.code;
          this.barcodeWatcher = result.codeResult.code;
          test.innerHTML = `<h1>読み取られたコード: ${result.codeResult.code}</h1>`;
        }
      });


    },
    loadCamera: function(camera) {

      const constrains = {
        audio: false,
        video: {
          width: 300,
          height: 200,
          // facingMode: { exact: "environment" }  // リアカメラを利用する場合
        }
      }
      navigator.mediaDevices.getUserMedia(constrains)
        .then(((stream)=>{
          camera.srcObject = stream;
          onloadedmetadata = () => { camera.play };
        })).catch(((err)=>{ console.log(err) }))

    },
    setUpBarcode(text) {
      // this.barcodeEvent(text);
      this.$emit('barcode', text);
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
#cameraDemo > video {
  position: relative;
}
.drawingBuffer {
  position: absolute;
  margin-left: -639px;
}
@media screen and (max-width: 480px) {
  .drawingBuffer {
    margin-top: -480px;
    margin-left: -193px;
  }
}
</style>
