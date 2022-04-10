<template>
  <q-page>
    <div class="bg-green-300 border-green-600 border-b p-4 m-4 rounded">
      Hello World
    </div>
    <video
      id="video"
      width="720"
      height="560"
      ref="video"
      autoplay
      muted
    ></video>
    <img width="100" height="100" ref="photo" />
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
  data: () => ({
    image: null,
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

    video.addEventListener("play", async () => {
      const labeledFaceDescriptors = await loadLabeledImages();
      const faceMatcher = new faceapi.FaceMatcher(labeledFaceDescriptors, 0.6);
      console.log("DO I GET HERE?!");
      const canvas = faceapi.createCanvasFromMedia(video);
      console.log("this is the canvas", canvas);
      document.body.append(canvas);
      const displaySize = { width: video.width, height: video.height };
      faceapi.matchDimensions(canvas, displaySize);
      setInterval(async () => {
        const imageBuffer = await component.takePicture(canvas);
        const image = await faceapi.fetchImage(imageBuffer);
        console.log("this is the image", image);
        const detections = await faceapi
          .detectAllFaces(image)
          .withFaceLandmarks()
          .withFaceDescriptors();
        console.log("this is the detections", detections);
        const resizedDetections = faceapi.resizeResults(
          detections,
          displaySize
        );
        console.log(displaySize)
        
        const results = resizedDetections.map((d) =>
          faceMatcher.findBestMatch(d.descriptor)
        );
        console.log("this is the results", results);
        if (results[0]) {
          await component.postToAuth(imageBuffer)
        }
        results.forEach((result, i) => {
          const box = resizedDetections[i].detection.box;
          const drawBox = new faceapi.draw.DrawBox(box, {
            label: result.toString(),
          });
          drawBox.draw(canvas);
        });
      }, 1000);
    });
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
     var myHeaders = new Headers();
      myHeaders.append("Content-Type", "application/json");
      console.log(imageBuffer)
      var raw = JSON.stringify({ image_buffer: imageBuffer });

      var requestOptions = {
        method: "POST",
        headers: myHeaders,
        body: raw,
        redirect: "follow",
      };

      const response = await fetch("http://localhost:3000/authenticate", requestOptions)
      const data = await response.json()
      console.log("this is the data", data)
      window.alert(data.message)
    },
  },
});
</script>
