<template>
  <q-page>
    <q-banner class="bg-primary text-white">
     <h2>Pay with Eat.eth</h2>
      <template v-slot:action>
        <q-btn @click="authenticate()">Checkout with EatEth Id</q-btn>
      </template>
    </q-banner>
    <q-input v-model="firstName" label="what is your first name"></q-input>
    <q-input v-model="lastName" label="what is your last name"></q-input>
    <video
      id="video"
      width="720"
      height="560"
      ref="video"
      autoplay
      muted
    ></video>
    <img width="100" height="100" ref="photo" />
    <q-dialog v-model="card">
      <q-card class="my-card">
        <q-img
          src="https://cdn.glitch.global/bdb630b5-11ba-4505-a157-8b517abc9ed1/EatEthDesign%20(2).png?v=1649622094358"
        />

        <q-card-section>
          <q-btn
            fab
            color="primary"
            icon="payments"
            class="absolute"
            style="top: 0; right: 12px; transform: translateY(-50%)"
          />

          <div class="row no-wrap items-center">
            <div class="col text-h6 ellipsis">
              Receipt for
              <q-list v-for="(item, index) of groceryList" v-bind:key="index">
                <q-item clickable v-ripple>
                  <q-item-section>{{item.name}}</q-item-section>
                </q-item>
              </q-list>
            </div>
            <div
              class="
                col-auto
                text-grey text-caption
                q-pt-md
                row
                no-wrap
                items-center
              "
            >
              <q-icon name="paid" />
            </div>
          </div>

          <q-rating v-model="stars" :max="5" size="32px" />
        </q-card-section>

        <q-card-section class="q-pt-none">
          <div class="text-subtitle1">Receipt:</div>
          <div class="text-caption text-grey">
            {{ receipt }}
          </div>
        </q-card-section>

        <q-separator />

        <q-card-actions align="right">
          <q-btn v-close-popup flat color="primary" label="close" />
          <q-btn v-close-popup flat color="primary" round icon="paid" />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script>
import { defineComponent } from "vue";
import * as faceapi from "face-api.js";

import { ref } from "vue";

export default defineComponent({
  name: "IndexPage",
  setup() {
    return {
      file: ref(null),
    };
  },
  computed: {
    checkoutUser() {
      if (!this.firstName) return undefined;
      if (!this.lastName) return undefined;
      if (this.firstName && this.lastName)
        return `${this.firstName}_${this.lastName}`;
      return undefined;
    },
  },
  props: {
    groceryList: Array,
  },
  data: () => ({
    image: null,
    firstName: null,
    lastName: null,
    authenticated: false,
    card: false,
    receipt: null,
  }),
  mounted() {
    const component = this;
    console.log("this is the video", this.$refs.video);

    const video = this.$refs.video;

    Promise.all([
      faceapi.nets.tinyFaceDetector.loadFromUri("/models"),
      faceapi.nets.faceLandmark68Net.loadFromUri("/models"),
      faceapi.nets.faceRecognitionNet.loadFromUri("/models"),
      faceapi.nets.faceExpressionNet.loadFromUri("/models"),
      faceapi.nets.faceRecognitionNet.loadFromUri("/models"),
      faceapi.nets.faceLandmark68Net.loadFromUri("/models"),
      faceapi.nets.ssdMobilenetv1.loadFromUri("/models"),
    ]).then(startVideo);

    function startVideo() {
      navigator.getUserMedia(
        { video: {} },
        (stream) => (video.srcObject = stream),
        (err) => console.error(err)
      );
    }

    function loadLabeledImages() {
      const labels = [
        "Black Widow",
        "Captain America",
        "Captain Marvel",
        "Hawkeye",
        "Jim Rhodes",
        "Thor",
        "Tony Stark",
        "Andrew",
      ];
      return Promise.all(
        labels.map(async (label) => {
          const descriptions = [];
          for (let i = 1; i <= 2; i++) {
            const img = await faceapi.fetchImage(
              `http://localhost:8080/labeled_images/${label}/${i}.jpg`
            );
            const detections = await faceapi
              .detectSingleFace(img)
              .withFaceLandmarks()
              .withFaceDescriptor();
            console.log(detections);
            descriptions.push(detections.descriptor);
            if (label == "Andrew") {
              console.log(detections.descriptor);
            }
          }

          return new faceapi.LabeledFaceDescriptors(label, descriptions);
        })
      );
    }

    console.log(video);

    video.addEventListener("play", async () => {});
  },
  methods: {
    async uploadImage(event) {
      console.log("this is the event", event);
      console.log(this.$refs.file.files);
      const file = this.$refs.file.files[0];
      // console.log("this is the refs", this.image);
      // const imageBuffer = await Buffer.from(this.image)
      start(file);
    },
    async authenticate() {
      const component = this;
      const video = this.$refs.video;
      const canvas = faceapi.createCanvasFromMedia(video);
      console.log("this is the canvas", canvas);
      const imageBuffer = await component.takePicture(canvas);
      const result = await component.postToAuth(imageBuffer);
      console.log(result);
    },
    async takePicture(canvas) {
      var context = canvas.getContext("2d");
      canvas.width = 300;
      canvas.height = 300;
      const video = this.$refs.video;
      context.drawImage(video, 0, 0, 300, 300);
      const photo = this.$refs.photo;
      var data = canvas.toDataURL("image/png");
      console.log("canvas data", data);
      photo.setAttribute("src", data);
      return data;
    },
    async postToAuth(imageBuffer) {
      const video = this.$refs.video;
      if (this.authenticated) return;
      var myHeaders = new Headers();
      myHeaders.append("Content-Type", "application/json");
      console.log(imageBuffer);
      var raw = JSON.stringify({
        image_buffer: imageBuffer,
        label: this.checkoutUser,
      });

      var requestOptions = {
        method: "POST",
        headers: myHeaders,
        body: raw,
        redirect: "follow",
      };

      const response = await fetch(
        "http://localhost:3000/authenticate",
        requestOptions
      );
      const data = await response.json();
      console.log("this is the data", data);
      this.authenticated = data.authenticated;
      if (this.authenticated) {
        this.card = true;
        this.receipt = JSON.stringify(data.receipt);
        var mediaStream = video.captureStream();
        console.log(mediaStream);
        mediaStream.getTracks().forEach(function (track) {
          track.stop();
        });
      }
    },
  },
});
</script>
