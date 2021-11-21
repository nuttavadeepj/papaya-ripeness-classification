<template>
  <v-app>
    <div class="d-flex align-center">
      <router-link to="/home">
        <v-img
          alt="Vuetify Logo"
          class="shrink mt-10 pl-10"
          contain
          src="../public/papayaya.png"
          width="300"
        />
      </router-link>
    </div>

    <div class="text-center" id="text">
      Let me check! <br />
      Your papaya is ripe or not
    </div>

    <div class="box_mas">
      <div class="mascot">
        <v-img src="../public/mascot.png" />
      </div>
      <div>
        <div v-if="image != null">
          <div class="result_box">
            <img :src="image" width="200" height="200" />
            <div class="text-center result_text">This papaya is <br> {{result}}</div>
          </div>
        </div>
        
      </div>
    </div>

    <div class="box_mas">
      <v-btn x-large
        ><v-file-input
          class="pa-0"
          hide-input
          small-chips
          truncate-length="32"
          @change="uploadImage"
        ></v-file-input>
        Upload
      </v-btn>
    </div>
  </v-app>
</template>

<script>
import * as tf from "@tensorflow/tfjs";
import { loadGraphModel } from "@tensorflow/tfjs-converter";
class L2 {
  static className = "L2";

  constructor(config) {
    return tf.regularizers.l1l2(config);
  }
}
export default {
  name: "App",
  data: () => ({
    model: null,
    image: null,
    result: null,
  }),
  mounted() {
    this.loadModel();
  },
  methods: {
    async loadModel() {
      console.log("init");
      tf.serialization.registerClass(L2);
      this.model = await loadGraphModel("shared/model/model.json");
      console.log("end");
    },
    prediction() {
      var image = new Image();
      image.src = this.image;
      image.onload = () => {
        const input = tf.browser.fromPixels(image);
        const inputResized = tf.image.resizeBilinear(input, [180, 180]);
        const predictionResult = this.model
          .predict(inputResized.reshape([-1, 180, 180, 3]))
          .softmax();
        const values = predictionResult.dataSync();
        const arr = Array.from(values);
        console.log(arr);
        let i = arr.indexOf(Math.max(...arr));
        let classlist = ["Medium", "Ripe", "Unripe"];
        this.result = classlist[i];

      };
    },
    uploadImage(e) {
      console.log(e);
      var reader = new FileReader();
      reader.onloadend = () => {
        if (reader.result) {
          this.image = reader.result;
        }
      };
      reader.readAsDataURL(e);
    },
  },
  watch: {
    image() {
      if (this.image != null) {
        this.prediction();
      }
    },
  },
};
</script>

<style >
@import url("https://fonts.googleapis.com/css2?family=Orbitron:wght@500&display=swap");

#app {
  font-family: "Orbitron", sans-serif;
  background: url("../public/space_bg.png") no-repeat center center !important;
  background-size: cover;
}
#text {
  color: aliceblue;
  font-size: 40px;
  background-color: black;
}
.result_img {
  width: 100px;
}
.result_text{
  color: black;
  font-size: 30px;
  padding-left: 30px;
}
.result_box {
  background-color: white;
  width: 550px;
  height: 250px;
  border-radius: 20px;
  padding-left: 40px;
  display: flex;
  align-items: center;
}
.mascot {
  width: 400px;
  margin-top: 30px;
}
.box_mas {
  display: flex;
  justify-content: center;
  align-items: center;
}
.btn_text {
  font-size: 300px;
  font-weight: 100;
}
</style>
