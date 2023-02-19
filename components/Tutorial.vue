<template>
  <div class="h-screen w-screen fixed top-0 px-2">
    <div
      ref="video-container"
      class="relative border-[3px] border-teal-400 rounded-xl h-[65vh]"
    >
      <video id="video" playsinline class="w-screen h-full">
        Video stream not available.
      </video>
      <button
        id="startbutton"
        class="absolute bottom-0 left-1/2 translate-x-[-50%] w-[65px] h-[65px] flex justify-center items-center font-medium text-sm bg-black rounded-full text-white translate-y-[30px] after:absolute after:w-[60px] after:h-[60px] after:content-[''] after:rounded-full after:border-2 after:border-teal-400"
        @click="takePicture"
      >
        {{ activeIndex < 7 ? stages[activeIndex].text : "Submit" }}
      </button>
    </div>
    <div class="flex flex-col justify-end border border-green-400 h-[25vh]">
      <canvas id="canvas"> </canvas>
      <div class="output flex gap-4">
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
</template>

<script>
export default {
  name: "NuxtTutorial",
  data() {
    return {
      width: 0,
      height: 0,
      video: null,
      photo: null,
      canvas: null,
      isStreaming: false,
      stream: null,
      startInspection: false,
      activeIndex: 0,
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
    };
  },
  mounted() {
    // if (this.check) {
    const videoContainer = this.$refs["video-container"];
    this.width = videoContainer.clientWidth;
    this.height = videoContainer.clientHeight;
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
        return (this.startInspection = true);
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
          fetch(data)
            .then((res) => res.blob())
            .then((data) => {
              console.log(data, "image");
              photo.setAttribute("src", data);
              this.images.push({
                [this.stages[this.activeIndex].tag]: data,
              });
              ++this.activeIndex;
            })
            .catch((err) => console.log(err, "error"));
        } else {
          this.clearPhoto();
        }
      } else {
        alert("Inspection completed!!! ");
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
  },
  computed: {},
  watch: {
    // stream(val) {
    //   console.log(val, "stream");
    // },
    activeIndex(val) {
      console.log(val, "active Index");
    },
    images(val) {
      console.log(val, "images");
    },
  },
};
</script>

<style>
#video {
  /*width: v-bind(width);
  height: v-bind(height);*/
}

#photo {
  border: 1px solid black;
  box-shadow: 2px 2px 3px black;
  width: 320px;
  height: 240px;
}

#canvas {
  display: none;
}

.output {
  width: 340px;
  vertical-align: top;
}
</style>
<!-- <svg class="position-absolute"
               style="height: 100px; transform: rotate(270deg); width: 100px;">
            <circle
              class=""
              stroke-width="4"
              stroke="transparent"
              fill="transparent"
              r="25"
              cx="50"
              cy="50"
            />
            <circle
              :style="{color: progress.color}"
              stroke-width="4"
              :stroke-dasharray="circumference"
              :stroke-dashoffset="offset"
              stroke-linecap="round"
              stroke="currentColor"
              fill="transparent"
              r="25"
              cx="50"
              cy="50"
            />
          </svg> -->
