<template>
  <div
    v-if="activeStep === 'inspect'"
    class="h-screen w-screen fixed overflow-hidden top-0"
  >
    <div ref="video-container" class="bg-black">
      <video id="video" playsinline class="w-screen h-screen">
        Video stream not available.
      </video>
    </div>

    <!-- camera view -->
    <div
      ref="camera-view"
      class="w-[97%] mx-auto absolute top-0 left-2 h-[60vh] border-2 z-[20] border-transparent rounded-lg camera-view"
    ></div>

    <!-- Timer -->
    <button
      class="absolute top-3 left-4 w-[42px] h-[42px] rounded-full bg-white border-[3px]"
      :class="[
        activeIndex > 0 && timeLeft < 15
          ? 'border-red-600'
          : activeIndex > 0 && timeLeft <= 30
          ? 'border-amber-300'
          : activeIndex > 0 && timeLeft > 15
          ? 'border-green-600'
          : 'border-black',
      ]"
    >
      <span
        class="text-sm font-medium"
        :class="[
          activeIndex > 0 && timeLeft <= 15
            ? 'text-red-600'
            : activeIndex > 0 && timeLeft <= 30
            ? 'text-amber-300'
            : activeIndex > 0 && timeLeft > 15
            ? 'text-green-600'
            : 'text-black',
        ]"
        >{{ timeLeft }}</span
      >
    </button>

    <!-- Step indicator -->
    <div
      class="absolute top-8 right-4 z-[49] bg-[#2b2b2b80] px-1 pt-2 pb-10 w-auto h-auto rounded-[1rem]"
    >
      <span
        class="flex justify-center items-center w-[23px] h-[23px] rounded-full mb-3 last:mb-0"
        :class="[activeIndex > idx + 1 ? 'bg-[#12B76A]' : 'bg-[#ffffff59]']"
        v-for="(svg, idx) in 6"
        :key="svg"
      >
        <svg
          width="12"
          height="8"
          viewBox="0 0 14 10"
          fill="none"
          xmlns="http://www.w3.org/2000/svg"
        >
          <path
            d="M12.2557 1.25391L4.52008 8.98949L1.00391 5.47332"
            stroke="white"
            stroke-width="1.68776"
            stroke-linecap="round"
            stroke-linejoin="round"
          />
        </svg>
      </span>
    </div>

    <canvas
      id="canvas"
      class="hidden absolute top-0 left-0 mx-auto translate-x-[2%] border-2 border-amber-300"
    >
    </canvas>

    <div
      class="flex flex-col justify-center fixed top-[60px] left-6 h-[178px] w-[176px] bg-[#05030366] rounded-lg"
    >
      <img
        :src="activeIndex < 7 ? stages[activeIndex].url : '/images/front.png'"
        alt="car-right"
        class="w-[145px] h-[114px] object-contain object-center block mx-auto"
      />
      <p class="text-sm text-center text-white">
        {{ activeIndex < 7 ? stages[activeIndex].desc : "All done submit now" }}
      </p>
    </div>

    <!-- CTA  -->
    <div
      ref="cta"
      class="flex flex-col justify-end fixed bottom-0 left-0 w-screen py-4 z-50 bg-transparent h-[28vh]"
    >
      <button
        id="startbutton"
        class="absolute select-none top-0 left-1/2 translate-x-[-50%] w-[65px] h-[65px] flex justify-center items-center border-white text-black font-medium text-sm bg-white rounded-full translate-y-[50%] after:absolute after:w-[57px] after:h-[57px] after:content-[''] after:rounded-full after:border after:border-[#101828]"
        @click="takePicture"
      >
        {{ activeIndex < 7 ? stages[activeIndex].text : "Submit" }}
      </button>

      <!-- <canvas id="canvas" class="hidden"> </canvas> -->

      <div class="output">
        <img
          id="photo"
          class="hidden"
          alt="The screen capture will appear in this box."
        />
      </div>
      <nuxt-link to="/" v-if="activeIndex === 0" class="text-red-600 text-center"
        >Cancel</nuxt-link
      >

      <!-- <div class="">
        <img
          :src="activeIndex < 7 ? stages[activeIndex].url : '/images/car-right.png'"
          alt="car-right"
          class="w-[150px] block mx-auto"
        />
        <p class="text-sm text-center text-white">
          {{ activeIndex < 7 ? stages[activeIndex].desc : "All done submit now" }}
        </p>
        <p class="text-red-600 text-center">Cancel</p>
      </div> -->
    </div>
  </div>

  <!-- View images -->
  <div v-else class="flex flex-col p-6 h-[93vh] fixed overflow-hidden">
    <p class="font-semibold text-center text-lg">Inspection Summary</p>
    <div class="grid grid-cols-2 gap-4 flex-1 overflow-y-auto mt-4">
      <div
        class="mb-4 last:mb-0 bg-[#F8FCFB] px-2"
        v-for="(car, idx) in images"
        :key="idx"
      >
        <img
          :src="car.imgPreview"
          alt="car-preview"
          class="block w-full h-auto aspect-square"
        />
        <p class="text-center text-sm capitalize">{{ car.title.replaceAll("_", " ") }}</p>
      </div>
    </div>
    <video :src="videoFile" class="w-full h-[200px]" controls></video>

    <div class="flex flex-col py-3 min-h-[130px]">
      <button class="h-[56px] font-medium w-full bg-teal-500 rounded-lg text-white">
        <nuxt-link to="/">Confirm Inspection</nuxt-link>
      </button>

      <button class="h-[56px] font-medium w-full text-teal-500" @click="reset">
        Inspect again
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: "NuxtTutorial",
  data() {
    return {
      activeStep: "inspect",
      lat: 0,
      long: 0,
      width: 0,
      height: 0,
      video: null,
      photo: null,
      canvas: null,
      isStreaming: false,
      stream: null,
      startInspection: false,
      activeIndex: 0,
      interval: null,
      timer: 90000,
      images: [],
      deniedLocation: false,
      deniedCamera: false,
      stages: [
        {
          text: "Start",
          tag: "right_image",
          url: "/images/start.png",
          desc: "",
        },
        {
          text: "",
          tag: "front_image",
          url: "/images/front.png",
          desc: "Front View ",
        },
        {
          text: "",
          tag: "right_image",
          url: "/images/right.png",
          desc: "Right Side",
        },
        {
          text: "",
          tag: "back_image",
          url: "/images/back.png",
          desc: "Back View",
        },
        {
          text: "",
          tag: "left_image",
          url: "/images/left.png",
          desc: "Left Side",
        },
        {
          text: "",
          tag: "dashboard_image",
          url: "/images/front.png",
          desc: "Dashboard View",
        },
        {
          text: "Finish",
          tag: "interior_back",
          url: "/images/back.png",
          desc: "Back Seat",
        },
      ],
      img: "",
      cameraViewHeight: 0,
      mediaRecorder: null,
      videoChunks: [],
      videoBlob: null,
      videoFile: null,
      navigator: null,
    };
  },
  mounted() {
    this.setCameraView();

    this.getGeoLocation()
      .then(() => this.initMedia())
      .catch((err) => {
        console.error(err, "user denied");
      });
  },
  methods: {
    check() {
      if (window.self !== window.top) {
        // Ensure that if our document is in a frame, we get the user
        // to first open it in its own tab or window. Otherwise, it
        // won't be able to request permission for camera access.
        document.querySelector(".contentarea").remove();
        const button = document.createElement("button");
        button.textContent = "View live result of the example code above";
        document.body.append(button);
        button.addEventListener("click", () => window.open(location.href));
        return true;
      }
      return false;
    },
    setCameraView() {
      const deviceHeight = window.innerHeight;
      const cameraView = this.$refs["camera-view"];
      //   const ctaView = this.$refs["cta"];
      this.width = cameraView.clientWidth;
      //   this.height = deviceHeight - ctaView.clientHeight;
      //   this.cameraViewHeight = deviceHeight - (ctaView.clientHeight + 10);
      this.height = deviceHeight;
      this.cameraViewHeight = deviceHeight;

      //   test
      const canvas = document.getElementById("canvas");
      canvas.width = this.width;
      canvas.height = this.height;
    },
    getGeoLocation() {
      return new Promise((resolve, reject) => {
        try {
          if ("geolocation" in navigator) {
            this.navigator = navigator.geolocation.watchPosition(
              (cord) => this.setGeoLocation(cord, resolve),
              (err) => this.catchGeoLocationError(err, reject)
            );
          } else {
            console.error("GPS not available on device");
          }
        } catch (e) {
          console.error(e);
        }
      });
    },
    setGeoLocation(position, resolve) {
      this.lat = position.coords.latitude;
      this.long = position.coords.longitude;
      resolve("Permission allowed 😁");
    },
    catchGeoLocationError(err, reject) {
      console.error(err, "error");
      reject("user denied permission 🤨");
    },
    initMedia() {
      const video = document.getElementById("video");
      this.video = video;
      const constraints = {
        audio: false,
        video: {
          width: { ideal: 1280 },
          height: { ideal: 720 },
          // facingMode: { exact: "environment" },
        },
      };
      try {
        navigator.mediaDevices
          .getUserMedia(constraints)
          .then((stream) => {
            this.stream = stream;
            this.startVideo(stream, video);
            this.recordVideo(stream);
          })
          .catch((err) => {
            console.error(err, "user denied");
            this.deniedCamera = true;
          });
      } catch (error) {
        console.log("get user again 😡");
      }
    },
    recordVideo(stream) {
      let ctx = this;
      ctx.mediaRecorder = new MediaRecorder(stream);
      if (!ctx?.mediaRecorder) {
        console.log(ctx?.mediaRecorder, "ctx.mediaRecorder");
        alert("supported!!!");
        return null;
      }
      ctx.mediaRecorder.ondataavailable = (event) => {
        ctx.videoChunks.push(event.data);
      };
      ctx.mediaRecorder.onstop = (event) => {
        const blob = new Blob(ctx.videoChunks, {
          type: "video/mp4",
        });
        const url = URL.createObjectURL(blob);
        ctx.videoBlob = blob;
        ctx.videoFile = url;
      };
    },
    startVideo(stream, video) {
      this.canvas = document.getElementById("canvas");
      video.srcObject = stream;
      video.play();

      this.video.addEventListener(
        "canplay",
        () => {
          if (!this.isStreaming) {
            this.isStreaming = true;
          }
        },
        false
      );
    },
    takePicture() {
      if (!this.startInspection) {
        ++this.activeIndex;
        this.startInspection = true;
        this.startTimer();
        this.mediaRecorder?.start();
        return null;
      }

      if (this.activeIndex < 7) {
        const photo = document.getElementById("photo");
        this.photo = photo;
        const canvas = document.getElementById("canvas");
        this.canvas = canvas;
        const video = document.querySelector("#video");

        const context = canvas.getContext("2d");
        if (this.width && this.height) {
          canvas.width = this.width;
          canvas.height = this.height;
          context.drawImage(video, 0, 0, this.width, this.height);
          const data = canvas.toDataURL("image/png");
          this.img = data;
          fetch(data)
            .then((res) => res.blob())
            .then((blob) => {
              photo.setAttribute("src", blob);
              this.images.push({
                title: this.stages[this.activeIndex].tag,
                blob: blob,
                imgPreview: data,
              });
              ++this.activeIndex;
            })
            .catch((err) => console.log(err, "error"));
        } else {
          this.clearPhoto();
        }
      } else {
        this.activeStep = "preview";
        this.clearInterval();
        this.mediaRecorder?.stop();
      }
    },
    clearPhoto() {
      this.photo = document.getElementById("photo");
      const context = this.canvas.getContext("2d");

      context.fillStyle = "#AAA";
      context.fillRect(0, 0, this.canvas.width, this.canvas.height);

      const data = canvas.toDataURL("image/png");
      this.photo.setAttribute("src", data);
    },
    stopRecording() {
      this.stream.getTracks().forEach((track) => track.stop());
    },
    startTimer() {
      this.interval = setInterval(() => {
        this.timer = this.timer - 1000;
      }, 1000);
    },
    clearInterval() {
      clearInterval(this.interval);
    },
    reset() {
      this.activeStep = "inspect";
      this.startInspection = false;
      this.activeIndex = 0;
      this.interval = null;
      this.timer = 90000;
      this.images = [];
      setTimeout(() => {
        this.initMedia();
        this.setCameraView();
      }, 1000);
    },
  },
  destroyed() {
    this.stopRecording();
    this.clearInterval();
    navigator.geolocation.clearWatch(this.navigator);
  },
  computed: {
    ct() {
      return `${this.cameraViewHeight}px`;
    },
    timeLeft() {
      return this.timer / 1000;
    },
  },
  watch: {
    timer(val) {
      if (val <= 0) {
        this.timer = 0;
        this.clearInterval();
      }
    },
    videoBlob(val) {
      console.log(val, "video blob");
    },
  },
};
</script>
