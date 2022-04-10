<template>
  <div class="q-pa-md" style="max-width: 100em">
    <q-stepper v-model="step" vertical color="primary" animated>
      <q-step
        :name="1"
        title="Add Personal Information"
        icon="settings"
        :done="step > 1"
      >
        <q-form
          ref="form"
          @submit="onSubmit"
          @reset="onReset"
          class="q-gutter-md"
        >
          <q-input
            filled
            v-model="firstName"
            label="Your name *"
            hint="Name and surname"
            lazy-rules
            :rules="[
              (val) => (val && val.length > 0) || 'Please type something',
            ]"
          />

          <q-input
            filled
            v-model="lastName"
            label="Your name *"
            hint="Name and surname"
            lazy-rules
            :rules="[
              (val) => (val && val.length > 0) || 'Please type something',
            ]"
          />

          <q-input
            filled
            type="number"
            v-model="age"
            label="Your age *"
            lazy-rules
            :rules="[
              (val) => (val !== null && val !== '') || 'Please type your age',
              (val) => (val > 0 && val < 100) || 'Please type a real age',
            ]"
          />
          <q-input
            filled
            v-model="guardiansName"
            label="Your parent or Guardian's Name"
            hint="Your parent or Guardian's Name"
            lazy-rules
            :rules="[
              (val) => (val && val.length > 0) || 'Please type something',
            ]"
          />

          <q-toggle v-model="accept" label="I agree to the terms" />

          <div>
            <q-btn label="Submit" type="submit" color="primary" />
            <q-btn
              label="Reset"
              type="reset"
              color="primary"
              flat
              class="q-ml-sm"
            />
          </div>
        </q-form>
        <q-stepper-navigation>
          <q-btn @click="step = 2" color="primary" label="Continue" />
        </q-stepper-navigation>
      </q-step>
      <q-step
        :name="2"
        title="Register Your Face"
        caption="Optional"
        icon="create_new_folder"
        :done="step > 2"
      >
        <q-banner class="bg-primary text-white">
          Start Capture
          <template v-slot:action>
            <q-btn @click="startCapture()">Turn on Camera</q-btn>
            <q-btn @click="takePicture()">Take Picture</q-btn>
          </template>
        </q-banner>
        <div class="row">
          <div class="col-10">
            <video width="720" height="560" ref="video" autoplay muted></video>
          </div>
          <div class="col-2">
            Pictures Needed: 2 Pictures Taken: {{ pictures }}
          </div>
        </div>
        <q-stepper-navigation>
          <q-btn @click="step = 4" color="primary" label="Continue" />
          <q-btn
            flat
            @click="step = 1"
            color="primary"
            label="Back"
            class="q-ml-sm"
          />
        </q-stepper-navigation>
      </q-step>
    </q-stepper>
  </div>
</template>

<script>
import { initializeApp } from "firebase/app";
import { getStorage, uploadString } from "firebase/storage";
import { ref as fStorageRef } from "firebase/storage";
import { getFirestore } from "firebase/firestore";

import { getAnalytics } from "firebase/analytics";

// TODO: Add SDKs for Firebase products that you want to use
// https://firebase.google.com/docs/web/setup#available-libraries

// Your web app's Firebase configuration
// For Firebase JS SDK v7.20.0 and later, measurementId is optional

// Initialize Firebase
// const app = initializeApp(firebaseConfig);
// const analytics = getAnalytics(app);

import { useQuasar } from "quasar";
import { ref } from "vue";
const $q = useQuasar();
import * as faceapi from "face-api.js";

export default {
  data: () => ({
    firstName: null,
    lastName: null,
    age: null,
    accept: false,
    video: null,
    guardiansName: null,
    step: 1,
    app: null,
    storage: null,
    db: null,
    pictures: 0,
  }),
  watch: {
    step(val) {
      if (val != 2 && this.video) {
        console.log(this.video);
        this.video = null;
      }
    },
  },
  async mounted() {
    var requestOptions = {
      referrerPolicy: "strict-origin-when-cross-origin",
      body: null,
      method: "GET",
      mode: "cors",
      credentials: "include",
    };

    const response = await fetch(
      "http://localhost:8080/firebaseValues.json",
      requestOptions
    );
    console.log("this is the response", response);
    const firebaseConfig = await response.json();
    console.log(firebaseConfig);
    this.app = await initializeApp(firebaseConfig);
    this.db = await getFirestore(this.app);
    this.storage = await getStorage(this.app);
    console.log(this.app);

    // var data = await response.json()
    // console.log('this is the request', data)
  },
  methods: {
    onSubmit() {
      if (accept.value !== true) {
        $q.notify({
          color: "red-5",
          textColor: "white",
          icon: "warning",
          message: "You need to accept the license and terms first",
        });
      } else {
        $q.notify({
          color: "green-4",
          textColor: "white",
          icon: "cloud_done",
          message: "Submitted",
        });
      }
    },
    startCapture() {
      const component = this;
      const video = this.$refs.video;

      console.log(video);
      navigator.mediaDevices.getUserMedia({ video: true }).then((stream) => {
        video.srcObject = stream;
        component.video = stream;
      });
      video.addEventListener("play", async () => {
        const canvas = faceapi.createCanvasFromMedia(video);
        console.log("this is the canvas", canvas);
      });
    },
    onReset() {
      this.firstName = null;
      this.lastName = null;
      this.guardiansName = null;
      this.age = null;
      this.accept = false;
    },
    async takePicture() {
      const video = this.$refs.video;
      const canvas = faceapi.createCanvasFromMedia(video);

      var context = canvas.getContext("2d");
      canvas.width = 300;
      canvas.height = 300;
      context.drawImage(video, 0, 0, 300, 300);
      // const photo = this.$refs.photo
      var data = canvas.toDataURL("image/png");
      console.log("canvas data", data);
      const userImagerReference = fStorageRef(
        this.storage,
        `labeled_images/${this.firstName}_${this.lastName}/${
          this.pictures + 1
        }.jpg`
      );
      if (this.pictures < 2) {
        console.log("THIS IS THE USER", userImagerReference);
        await uploadString(userImagerReference, data, "data_url");
      } else {
        const folder = `labeled_images/${this.firstName}_${this.lastName}`;
        const newEnrolledChild = {
          firstName: this.firstName,
          lastName: this.lastName,
          guardianName: this.guardiansName,
          imageFolder: folder,
        };
        var myHeaders = new Headers();
        myHeaders.append("Content-Type", "application/json");

        var raw = JSON.stringify(newEnrolledChild);

        var requestOptions = {
          method: "POST",
          headers: myHeaders,
          body: raw,
          redirect: "follow",
        };

        const registerResult = await fetch("http://localhost:3000/register", requestOptions);
        const registerData = await registerResult.json()
        console.log("the register data", registerData)
      }

      this.pictures = this.pictures + 1;
      //photo.setAttribute('src', data);
      return data;
    },
  },
};
</script>
