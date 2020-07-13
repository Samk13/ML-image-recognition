<template>
  <div id="app" class="container mx-auto space-y-2">
    <loading class="bg-blue-200 text-xl font-black" v-if="!isModelLoaded"></loading>
    <div class="py-10 bg-teal-400 font-black text-4xl text-center rounded-lg">IMG -> AI -> GIF</div>
    <div class="bg-green-400 p-4 rounded-lg">
      <div class="flex w-full items-center justify-center">
        <label
          class="w-full flex flex-col items-center px-4 py-3 bg-white text-blue rounded-lg shadow-lg font-black uppercase border border-blue cursor-pointer hover:bg-blue hover:text-green-400"
        >
          <svg
            class="w-8 h-8"
            fill="currentColor"
            xmlns="http://www.w3.org/2000/svg"
            viewBox="0 0 20 20"
          >
            <path
              d="M16.88 9.1A4 4 0 0 1 16 17H5a5 5 0 0 1-1-9.9V7a3 3 0 0 1 4.52-2.59A4.98 4.98 0 0 1 17 8c0 .38-.04.74-.12 1.1zM11 11h3l-4-4-4 4h3v3h2v-3z"
            ></path>
          </svg>
          <span class="mt-2">Upload your image</span>
          <input type="file" class="hidden" name="uploadImage" @change="uploadImage">
        </label>
      </div>
    </div>

    <div class="md:flex bg-teal-400 rounded-lg p-4 space-x-4">
      <div class>
        <div class="w-full bg-white rounded-lg shadow-lg" v-show="img">
          <img ref="image" alt="Uploaded image" :src="img" class="w-screen">
          <div class="results">
            {{ result.label }}
            {{ result.confidence }}%
          </div>
        </div>
      </div>
      <div class="w-full bg-white rounded-lg shadow-lg" v-show="img">
        <div class>
          <img :src="result.giphy" :alt="result.alt">
          <p>{{result.alt }}</p>
        </div>
      </div>
    </div>
    <footer class="footer">
      <div class="content has-text-centered">
        <p>2020.</p>
      </div>
    </footer>
  </div>
</template>

<script>
import ml5 from "ml5";
import axios from "axios";

import Loading from "./components/Loading.vue";

export default {
  name: "App",
  components: {
    Loading
  },
  data() {
    return {
      classifier: {},
      isModelLoaded: false,
      result: {
        label: "",
        confidence: 0,
        giphy: "",
        giphyUrl: "",
        alt: ""
      },
      img: "",
      imgName: "Upload an image to start"
    };
  },
  mounted: function() {
    this.classifier = ml5.imageClassifier("MobileNet", this.modelLoaded);
  },
  methods: {
    modelLoaded: function() {
      console.log("Model Loaded!");
      this.isModelLoaded = true;
    },
    classify: function() {
      // https://github.com/ml5js/ml5-library/blob/development/src/utils/IMAGENET_CLASSES.js
      this.classifier.classify(this.$refs.image, (err, results) => {
        if (err) {
          console.error(err);
        } else {
          console.log(results);
          this.result.label = results[0].label;
          this.result.confidence = (results[0].confidence * 100).toFixed(2);
          this.getGiphy();
        }
      });
    },
    uploadImage: function(event) {
      const image = event.target.files[0];
      const reader = new FileReader();
      this.imgName = event.target.files[0].name;
      reader.readAsDataURL(image);
      reader.onload = e => {
        this.img = e.target.result;
        this.classify();
      };
    },
    getGiphy: function() {
      // https://developers.giphy.com/docs/sdk/#web
      const url =
        "https://api.giphy.com/v1/gifs/search?api_key=gnHx8iIxLiE3MLUDnVWJ6pcWDlI8LGqL&limit=1&q=";
      this.result.giphy = require("@/assets/timer.svg");
      this.result.alt = "loading gif";
      this.result.giphyUrl = "";
      axios
        .get(
          url +
            this.result.label +
            "&offset=" +
            Math.floor(Math.random() * 100 + 1)
        )
        .then(response => {
          const responseData = response.data.data[0];
          console.log(responseData);
          this.result.giphy = responseData.images.original.url;
          this.result.alt = responseData.title;
          this.result.giphyUrl = responseData.url;
        })
        .catch(e => {
          this.errors.push(e);
        });
    }
  }
};
</script>
