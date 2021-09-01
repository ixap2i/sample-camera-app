<template>
  <div>
    <div class='cameraSpace'>
      <div class='buttonGroup'>
        <button id='shutter' v-on:click='this.snapCamera()'>
          Cheese!
        </button>
        <button v-on:click='this.refreshPage()'>
          Refresh
        </button>
      </div>

      <div id='cameraDemo'>
      </div>
      <canvas id="canvas"></canvas>
    </div>

    <div id='appendField' @ocrTxt="$emit('ocrTxt', $event.target.value)">
    </div>

  </div>
</template>

<script>
export default {
  name: 'CameraPart',
  computed: {
    targetSplite: function() {
      return document.querySelector('#appendField');
    },
    pasteImage: function() {
      return 'test';
    },
  },
  props: {
    msg: String,
    barcode: String,
    url: {
      type: String,
      default: function() {
        return 'https://vision.googleapis.com/v1/images:annotate?key=AIzaSyD6zE1TyBncQGocEqknyy70oGUn7hmmxAY';
      }
    },
    headers: {
      type: Array,
      default: function() {
        return ['Content-Type', 'application/json'];
      }

    }

  },
  mounted: function() {

  },
  methods: {
    snapCamera: function() {
      const camera = document.getElementsByTagName('video')[0]
      // const canvas = document.querySelector('.drawingBuffer')
      const canvas = document.querySelector('#canvas')
      document.querySelector('#shutter').addEventListener('click', () => {
        const ctx = canvas.getContext('2d');

        camera.pause();
        setTimeout(() => {
          camera.play();
        }, 500)

        ctx.drawImage(camera, 0, 0, canvas.width, canvas.height);

        this.sendImage(canvas);
      });
    },
    sendImage: function(canvas) {
      //画像をBase64に変換する
      var sendImage = this.pasteSnappedImage(canvas);
      // const appendField = document.querySelector('#appendField');

      let body = {
        requests: [
          {image: {content: sendImage}, features: [{type: 'TEXT_DETECTION'}]}
        ]
      };
      const xhr = new XMLHttpRequest();
      xhr.open('POST', this.url, true);

      xhr.onreadystatechange = () => {

        const from_json = JSON.parse(xhr.responseText);
        const analizedText = from_json.responses[0].fullTextAnnotation.text;

        //---------------------------
        //CoronaLab 仮の健康度を算出 API実行
        const url2 = 'https://dev.lahmu-cdn-web.food-score.tech/api/v1/temporary_scores';
        let xhr2 = new XMLHttpRequest();
        xhr2.open('POST', url2, true );
        xhr2.setRequestHeader('Content-Type', 'application/json');

        let json = JSON.stringify({
          raw_material: "",
          raw_material_ocr_str: analizedText
        });

        xhr2.onreadystatechange = () => {
          const from_json2 = JSON.parse(xhr2.responseText);

          console.log(from_json2);

          this.targetSplite.innerHTML = from_json2.raw_material_ocr_str;
        };
        xhr2.send( json );
      };
      xhr.send( JSON.stringify(body) );
    },
    pasteSnappedImage: function(canvas) {
      var dataRaw = canvas.toDataURL("image/jpeg");
      var dataArray = dataRaw.split( ',' );
      var base64string = dataArray[ 1 ];
      return base64string;
    },
    setImage: function() {
      // this.targetSplite = from_json2.raw_material_ocr_str;
    },
    refreshPage: function() {
      location.reload();
    }
  }
}

</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.buttonGroup {
  display: flex;
  justify-content: space-evenly;
}
.cameraSpace {
  display: flex;
  justify-content: center;
}
#canvas {
  width: 640px;
  /* height: 480px; */
  border: solid black 1px;
}
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
@media screen and (max-width: 480px) {
  .cameraSpace { display: initial; }
}
</style>
