<template>
  <div id="app">
    <div style="position: relative; border: #fffa00 2px solid">
      <img :src="outline" />
      <v-easy-camera
        v-model="picture"
        ref="picpreview"
        :startOnMounted="true"
        output="blob"
      ></v-easy-camera>

      <div
        style="
          width: 100%;
          display: flex;
          align-items: center;
          justify-content: center;
          position: absolute;
          bottom: 0;
          left: 0;
        "
      >
        <button @click="cameraAction('start')">Start</button>
        <button @click="cameraAction('snap')">Snap</button>
        <button @click="cameraAction('stop')">Stop</button>
        <button @click="cameraAction('close')">Close</button>
      </div>
    </div>
  </div>
</template>

<script>
import EasyCamera from "easy-vue-camera";

export default {
  name: "App",
  components: {
    "v-easy-camera": EasyCamera,
  },
  data: function () {
    return {
      picture: "",
      outline: require('./assets/outline.svg')
    };
  },
  computed: {},
  methods: {
    cameraAction(opt) {
      if (opt === "start") {
        this.$refs.picpreview.start();
      } else if (opt === "snap") {
        this.$refs.picpreview.snap();

        setTimeout(() => {
          this.processFile(this.picture);
        }, 2000);
      } else if (opt === "stop") {
        this.$refs.picpreview.stop();
      } else if (opt === "close") {
        this.$refs.picpreview.close();
      }
    },
    processFile(blob) {
      // read the files
      var reader = new FileReader();

      if (blob instanceof Blob) {
        reader.readAsArrayBuffer(blob);

        reader.onload = function (event) {
          // blob stuff
          var blob = new Blob([event.target.result]); // create blob...
          window.URL = window.URL || window.webkitURL;
          var blobURL = window.URL.createObjectURL(blob); // and get it's URL

          // helper Image object
          var image = new Image();
          image.src = blobURL;
          image.onload = function () {
            // have to wait till it's loaded
            var canvas = document.createElement("canvas");
            var width = image.width;
            var height = image.height;

            // calculate the width and height, constraining the proportions
            if (width > height) {
              if (width > 100) {
                //height *= max_width / width;
                height = Math.round((height *= 100 / width));
                width = 100;
              }
            } else {
              if (height > 100) {
                //width *= max_height / height;
                width = Math.round((width *= 100 / height));
                height = 100;
              }
            }

            // resize the canvas and draw the image data into it
            canvas.width = width;
            canvas.height = height;
            var ctx = canvas.getContext("2d");
            ctx.drawImage(image, 0, 0, width, height);

            canvas.toBlob(function (blob) {
              const newImg = document.createElement("img");
              const url = URL.createObjectURL(blob);

              //send this blob to API

              newImg.onload = function () {
                // no longer need to read the blob so it's revoked
                URL.revokeObjectURL(url);
              };

              newImg.src = url;
              document.body.appendChild(newImg);
            });
          };
        };
      } else {
        // console.log(this.picture);
      }
    },
  },
};
</script>

<style>
html,
body {
  height: 100%;
  font-family: "Roboto";
  background: black;
}
</style>
