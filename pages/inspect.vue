<template>
  <div v-if="activeStep === 'inspect'" class="h-screen w-screen fixed top-0">
    <!-- <div v-if="img.length" class="fixed top-0 left-0 w-screen h-screen bg-white z-[1000]">
      <span class="absolute top-0 right-0 text-2xl" @click="img = ''">&#10005;</span>
      <img :src="img" alt="" class="w-full h-auto" />
    </div> -->
    <div ref="video-container" class="bg-black">
      <video id="video" playsinline class="w-screen min-h-screen h-full">
        Video stream not available.
      </video>
    </div>

    <!-- Step indicator -->
    <div
      ref="camera-view"
      class="w-[97%] mx-auto absolute top-0 left-2 h-[60vh] border-2 z-[20] border-teal-400 rounded-lg camera-view"
    ></div>

    <!-- Timer -->
    <button
      class="absolute top-3 left-4 w-[42px] h-[42px] rounded-full bg-white border-[3px]"
      :class="[
        activeIndex > 0 && timeLeft > 15
          ? 'border-green-600'
          : activeIndex > 0 && timeLeft < 15
          ? 'border-red-600'
          : 'border-black',
      ]"
    >
      <span
        class="text-sm font-medium"
        :class="[timeLeft < 15 ? 'text-red-600' : 'text-black']"
        >{{ timeLeft }}</span
      >
    </button>

    <div
      class="absolute top-8 right-4 z-[49] bg-[#2b2b2b80] px-1 py-2 w-auto h-auto rounded-lg"
    >
      <span
        class="flex justify-center items-center w-8 h-8 rounded-full mb-3 last:mb-0"
        :class="[activeIndex > idx + 1 ? 'bg-[#12B76A]' : 'bg-[#ffffff59]']"
        v-for="(svg, idx) in 6"
        :key="svg"
      >
        <svg
          width="14"
          height="10"
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

    <div
      ref="cta"
      class="flex flex-col justify-end fixed bottom-0 left-0 w-screen py-4 z-50 bg-white h-[28vh]"
    >
      <button
        id="startbutton"
        class="absolute top-0 left-1/2 translate-x-[-50%] w-[65px] h-[65px] flex justify-center items-center border-white font-medium text-sm bg-black rounded-full text-white translate-y-[-30px] after:absolute after:w-[60px] after:h-[60px] after:content-[''] after:rounded-full after:border-2 after:border-teal-400"
        @click="takePicture"
      >
        {{ activeIndex < 7 ? stages[activeIndex].text : "Submit" }}
      </button>

      <canvas id="canvas" class="hidden"> </canvas>

      <div class="output">
        <img
          id="photo"
          class="hidden"
          alt="The screen capture will appear in this box."
        />
      </div>

      <div class="">
        <img
          :src="activeIndex < 7 ? stages[activeIndex].url : '/images/car-right.png'"
          alt="car-right"
          class="w-[150px] block mx-auto"
        />
        <p class="text-sm text-center">
          {{ activeIndex < 7 ? stages[activeIndex].desc : "All done submit now" }}
        </p>
        <p class="text-red-600 text-center">Cancel</p>
      </div>
    </div>
  </div>

  <!-- View images -->
  <div v-else class="p-6">
    <div class="" v-for="(car, idx) in images" :key="idx">
      <img
        :src="car.imgPreview"
        alt="car-preview"
        class="block mb-4 w-full h-[200px] aspect-square"
      />
      <span class="text-center">{{ car.title.replaceAll("_", " ") }}</span>
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
      stages: [
        {
          text: "Start",
          tag: "right_image",
          url: "/images/car-right.png",
          desc: 'Click "start" to begin.',
        },
        {
          text: "Next",
          tag: "right_image",
          url: "/images/car-right.png",
          desc: "Right Side View of your car",
        },
        {
          text: "Next",
          tag: "front_image",
          url: "/images/car-front.png",
          desc: "Front View of your car with Plate No.",
        },
        {
          text: "Next",
          tag: "back_image",
          url: "/images/car-back.png",
          desc: "Back View of your car with Plate No.",
        },
        {
          text: "Next",
          tag: "left_image",
          url: "/images/car-left.png",
          desc: "Left Side View of your car",
        },
        {
          text: "Next",
          tag: "dashboard_image",
          url: "/images/car-dashboard.jpeg",
          desc: "Interior view of your car showing your vehicle dashboard",
        },
        {
          text: "Finish",
          tag: "interior_back",
          url: "/images/car-interior.jpeg",
          desc: "Interior view of your car showing your vehicle back seat",
        },
      ],
      img: "",
      cameraViewHeight: 0,
    };
  },
  mounted() {
    // if (this.check) {
    // const videoContainer = this.$refs["video-container"];
    const deviceHeight = window.innerHeight;
    const cameraView = this.$refs["camera-view"];
    const ctaView = this.$refs["cta"];
    this.width = cameraView.clientWidth;
    this.height = deviceHeight - ctaView.clientHeight;
    this.cameraViewHeight = deviceHeight - ctaView.clientHeight;

    this.getGeoLocation();
    this.initMedia();
    // }
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
    getGeoLocation() {
      try {
        if ("geolocation" in navigator) {
          navigator.geolocation.watchPosition((position) => {
            this.lat = position.coords.latitude;
            this.long = position.coords.longitude;
          });
        } else {
          console.error("GPS not available on device");
        }
      } catch (e) {
        console.error(e);
      }
    },
    initMedia() {
      const video = document.getElementById("video");
      this.video = video;
      const constraints = {
        audio: false,
        video: {
          width: { ideal: 1280 },
          height: { ideal: 720 },
          // frameRate: { ideal: 10, max: 15 },
          facingMode: { exact: "environment" },
        },
      };
      try {
        navigator.mediaDevices
          .getUserMedia(constraints)
          .then((stream) => {
            this.stream = stream;
            this.startVideo(stream, video);
          })
          .catch((err) => {
            console.error(err);
          });
      } catch (error) {
        console.log("get user again 😡");
      }
    },
    startVideo(stream, video) {
      this.canvas = document.getElementById("canvas");
      video.srcObject = stream;
      video.play();

      this.video.addEventListener(
        "canplay",
        () => {
          if (!this.isStreaming) {
            // this.height = this.video.videoHeight / (this.video.videoWidth / this.width);

            // this.video.setAttribute("width", this.width);
            // this.video.setAttribute("height", this.height);
            // this.canvas.setAttribute("width", this.width);
            // this.canvas.setAttribute("height", this.height);
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
          console.log(this.width, this.height);
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
        alert("Inspection completed!!! ");
        this.activeStep = "preview";
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
  },
  destroyed() {
    this.stopRecording();
    this.clearInterval();
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
    // stream(val) {
    //   console.log(val, "stream");
    // },
    // activeIndex(val) {
    //   console.log(val, "active Index");
    // },
    images(val) {
      console.log(val, "images");
    },
    timer(val) {
      if (val <= 0) {
        this.timer = 0;
        this.clearInterval();
      }
    },
  },
};
</script>

<style>
#photo {
  border: 1px solid black;
  box-shadow: 2px 2px 3px black;
  width: 320px;
  height: 240px;
}

.camera-view {
  height: v-bind(ct);
}
</style>
