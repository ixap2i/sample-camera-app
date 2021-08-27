<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <div class='cameraSpace'>
      <div id='cameraDemo'>
      </div>
      <canvas id="canvas"></canvas>
    </div>
    <button id='shutter' v-on:click='this.snapCamera()'>
      Cheese!
    </button>

    <div id='appendField' @ocrTxt="$emit('ocrTxt', $event.target.value)">
    </div>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  mounted: function() {

  },
  methods: {
    snapCamera: function() {
      const camera = document.getElementsByTagName('video')[0]
      const canvas = document.querySelector('.drawingBuffer')
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
      var dataRaw = canvas.toDataURL("image/jpeg");
      var dataArray = dataRaw.split( ',' );
      var base64string = dataArray[ 1 ];
      const appendField = document.querySelector('#appendField');
      // const div_raw_matrial = document.getElementById("raw_matrial");

      let body = {
        requests: [
          {image: {content: base64string}, features: [{type: 'TEXT_DETECTION'}]}
        ]
      };

      const url = 'https://vision.googleapis.com/v1/images:annotate?key=AIzaSyD6zE1TyBncQGocEqknyy70oGUn7hmmxAY';
      const xhr = new XMLHttpRequest();
      xhr.open('POST', url, true);
      xhr.setRequestHeader('Content-Type', 'application/json');
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
          appendField.innerHTML = from_json2.raw_material_ocr_str;
        };
        xhr2.send( json );
      };
      xhr.send( JSON.stringify(body) );
    }
  }
}

</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.hello {
  display: flex;
  flex-direction: column;
}
.cameraSpace {
  display: flex;
  justify-content: center;
}
#shutter {
  justify-content: center;
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
</style>
