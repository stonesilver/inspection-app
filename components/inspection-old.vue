<template>
    <div v-if="!done" id="screen" class="screen">
      <div class="video-bg camView position-relative" id="liveView">
        <div v-if="!isLoading" class="counter d-flex align-items-center justify-content-center rounded-circle">
          <svg class="position-absolute"
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
          </svg>
          <div :style="{color: progress.color}"
               class="bg-white rounded-circle d-flex align-items-center justify-content-center font-weight-bold text-14">
            {{
              `0:${time}`
            }}
          </div>
        </div>
  
        <div v-if="!isLoading" class="stages d-flex justify-content-center">
          <div>
            <div v-for="(stage, index) in stages.length"
                 :class="[streaming && activeItem >= index ? 'bg-green' : '', index < (stages.length -1) ? 'mb-2': '']"
                 class="stage rounded-circle d-flex align-items-center justify-content-center">
              <div>
                <img v-if="streaming && activeItem >= index" src="/images/icons/check.svg" class="img-fluid mt-n1"
                     alt="check">
                <img v-else src="/images/icons/check-white.svg" class="img-fluid mt-n1" alt="check-success">
              </div>
            </div>
          </div>
        </div>
  
        <div class="d-flex justify-content-center">
          <div class="text-20 font-weight-light text-center text-white loading" v-if="isLoading">Loading AI Engine...
          </div>
          <div class="ai-video-container d-flex justify-content-center align-items-center" :class="[isInside ? 'light': 'dark']" v-else>
            <div class="text-white" v-if="!isInside">Car image must fit within this box</div>
          </div>
          <video id="video" autoplay playsinline ref="video" class="video-player"/>
        </div>
  
        <div v-if="!streaming" class="director d-flex justify-content-center">
          <div v-if="!isLoading">
            <img src="/images/director.png" class="img-fluid" alt="">
            <div class="text-white text-center text-14 font-weight-medium mt-n2">Go in this direction</div>
          </div>
        </div>
      </div>
  
      <div class="action-pane position-relative" style="background-color: #FFFFFF !important">
        <div class="d-flex justify-content-center">
          <button type="button" @click="snap" :disabled="isLoading" v-if="!timeOut"
                  class="action-btn text-white text-capitalize">
            <span class="rim">
              {{ stages[activeItem] ? stages[activeItem].text : 'Start' }}
            </span>
          </button>
        </div>
        <div class="text-center mt-4 pt-3">
          <img :src="stages[activeItem] ? stages[activeItem].url : '/images/car-right.png'" class="img-fluid stage-image"
               alt="cars"/>
          <div class="mt-1 font-weight-medium text-center text-dark">
            {{ stages[activeItem] ? stages[activeItem].desc : 'Click "start" to begin.' }}
          </div>
        </div>
      </div>
  
      <canvas id="canvas">
        <div class="output">
          <img src="" id="photo" alt="The screen capture will appear in this box."/>
        </div>
      </canvas>
    </div>
  
    <div v-else class="container pb-5">
      <div class="d-flex justify-content-center">
        <div style="width: 50px; height: 50px"
             class="d-flex mb-3 rounded-circle mt-4 justify-content-center align-items-center bg-green">
          <div>
            <img src="/images/icons/check.svg" style="margin-top: -3px; width: 24px; height: auto"
                 alt="inspection success">
          </div>
        </div>
      </div>
  
      <div class="mb-3">
        <div class="text-gray-900 text-center">Completed !</div>
        <p class="font-weight-light text-12 text-center">Great! You have Completed Inspection, would you like to
          proceed?</p>
        <div class="text-center font-weight-bold text-dark h6">Preview</div>
      </div>
  
      <div class="row mt-4 scrollable-yy" style="flex-wrap: wrap; height: calc(100vh - 400px)">
        <div v-for="(image, key) in photos" class="col-lg-6 mb-3">
          <div style="background: #F3F4F6;" class="text-center">
            <img :src="image.url" class="img-fluid" alt="">
            <div class="text-capitalize py-2">{{ key.replace(/_/g, " ") }}</div>
          </div>
  
        </div>
      </div>
  
      <div class="w-100 my-4">
        <button type="button" @click="proceed" class="btn mb-2 btn-primary text-white font-weight-medium w-100">
          Continue
        </button>
        <button type="button" class="btn bg-transparent font-weight-medium text-green w-100" @click="resetProcess">
          Inspect again
        </button>
        <slot name="cancel-button"/>
      </div>
      <video id="preview" playsinline autoplay loop muted class="img-fluid"/>
    </div>
  </template>
  
  <script>
  
  export default {
    name: "inspect-vehicle",
    data() {
      return {
        done: false,
        activeItem: 0,
        stages: [
          {
            text: 'Start',
            tag: 'welcome-screen',
            desc: 'Click "start" to begin.',
            url: '/car-assets/car-front.png',
          },
          {
            text: 'Next',
            tag: 'right_image',
            url: '/car-assets/car-right.png',
            desc: 'Right View of your car'
          },
          {
            text: 'Next',
            tag: 'front_image',
            url: '/car-assets/car-front.png',
            desc: 'Front View of your car with Plate No.'
          },
          {
            text: 'Next',
            tag: 'back_image',
            url: '/car-assets/car-back.png',
            desc: 'Back View of your car with Plate No.'
          },
          {
            text: 'Next',
            tag: 'left_image',
            url: '/car-assets/car-left.png',
            desc: 'Left Side View of your car'
          },
          {
            text: 'Next',
            tag: 'dashboard_image',
            url: '/car-assets/car-dashboard.jpeg',
            desc: 'Interior view of your car showing your vehicle dashboard'
          },
          {
            text: 'Finish',
            tag: 'interior_back',
            url: '/car-assets/car-interior.jpeg',
            desc: 'Interior view of your car showing your vehicle back seat'
          },
        ],
        track: "",
        timeOut: false,
        stream: null,
        devices: null,
        err: null,
        recordedBlob: null,
        children: [],
        model: null,
        video: null,
        camera: "",
        canvas: null,
        photo: null,
        width: null,
        height: null,
        streaming: false,
        photos: {
          'right_image': {
            url: '',
            blob: null
          },
          'front_image': {
            url: '',
            blob: null
          },
          'back_image': {
            url: '',
            blob: null
          },
          'left_image': {
            url: '',
            blob: null
          },
          'dashboard_image': {
            url: '',
            blob: null
          },
          'interior_back': {
            url: '',
            blob: null
          },
        },
        recordedVideo: null,
        circumference: 80 * 2 * Math.PI,
        offset: 0,
        interval: null,
        lat: null,
        long: null,
        err2: [],
        time: 60,
        isLoading: false,
        isInside: false,
        progress: {
          color: '#05250a',
          high: 60,
          low: 0,
        },
      }
    },
    mounted() {
      this.getGeoLocation();
      this.getMedia();
    },
    methods: {
      getGeoLocation() {
        try {
          if ('geolocation' in navigator) {
            navigator.geolocation.watchPosition((position) => {
              this.lat = position.coords.latitude
              this.long = position.coords.longitude
            });
          } else {
            console.error('GPS not available on device')
          }
        } catch (e) {
          console.error(e)
        }
      },
      reset() {
        location.reload()
      },
      proceed() {
        this.$emit("done", this.photos);
      },
      resetProcess() {
        this.done = false;
        this.getGeoLocation();
        this.getMedia();
      },
      lockScreenOrientation() {
        window.screen.orientation.lock("portrait").then(res => {
          this.err = res
        }).catch(err => {
          this.err = err
        })
      },
      countDown() {
        const ctx = this;
        ctx.interval = setInterval(() => {
          --ctx.time
          if (ctx.time >= 40) ctx.progress.color = '#19b631'
          if (ctx.time < 40) ctx.progress.color = '#eca012'
          if (ctx.time < 20) ctx.progress.color = '#ec0505'
          ctx.offset -= 2.672
        }, 1000)
      },
      async snap() {
        if (this.activeItem <= 6) {
          if (!this.streaming) {
            this.isLoading = true;
            try {
              let recordedChunks = await this.startRecording(this.video.captureStream());
              this.recordedBlob = new Blob(recordedChunks, {type: "video/webm"});
              this.isLoading = false;
              await this.getMedia();
              this.streaming = true;
            } catch (e) {
              console.error(e)
            }
          }
          if (this.streaming) {
            this.takePicture();
            this.activeItem++
          }
        }
      },
      endCapturing() {
        this.stop(this.stream).then((e) => {
          if (e) {
            this.done = true;
            this.stream = null;
            this.streaming = false;
            this.activeItem = 0;
            this.timeOut = false;
            this.time = 60;
            this.offset = 0;
            this.canvas = null;
            this.streaming = false;
            this.progress.color = "#05250a"
            this.isLoading = false;
          }
        })
        clearInterval(this.interval);
      },
      async getCamera(stream, video) {
        const ctx = this;
        if (!navigator.mediaDevices?.enumerateDevices) {
          return console.error("enumerateDevices() not supported.");
        }
        //Close the camera if it has already been opened.
        try {
          await navigator.mediaDevices?.enumerateDevices().then(res => {
            let cam = res;
            //TODO handle case where there's no specified camera on the device i.e cam.length === 0
            if (cam.length === 0) {
              alert("Camera seems not to be supported or available at the moment")
              throw new Error('Camera not available')
            }
            if (ctx.camera.length < 1) {
              ctx.camera = cam[cam.length - 1].deviceId
              this.getMedia();
            } else {
              this.stream = stream;
              video.srcObject = stream;
              ctx.loading = true;
              video.addEventListener('loadeddata', () => {
                ctx.loading = false;
              });
            }
            return cam
          });
        } catch (e) {
          console.error(e)
        }
      },
      async startRecording() {
        if (!this.stream) {
          throw new Error("Stream is currently empty")
        }
        let recorder = new MediaRecorder(this.stream);
        let data = [];
        recorder.ondataavailable = event => data.push(event.data);
        recorder.start();
        this.countDown();
        let stopped = new Promise((resolve, reject) => {
          recorder.onstop = resolve;
          recorder.onerror = event => reject(event.name);
        });
        let recorded = this.wait(6000).then(() => recorder.state === "recording" && recorder.stop());
        return Promise.all([stopped, recorded]).then(() => data);
      },
      wait(delayInMS) {
        return new Promise(resolve => setTimeout(resolve, delayInMS));
      },
      stop(stream) {
        return new Promise((resolve, reject) => {
          try {
            stream.getTracks().forEach(track => {
              track.stop();
              track.enabled = false;
              if (track.readyState === 'ended') {
                resolve(true)
              }
  
            });
          } catch (e) {
            reject(e)
          }
        })
      },
      async getMedia() {
        const ctx = this;
        const video = document.querySelector('#video');
        this.video = video
        const constraints = {
          audio: false,
          video: {
            deviceId: ctx?.camera,
            width: {ideal: 1280},
            height: {ideal: 720},
            facingMode: {
              exact: 'environment'
            }
          }
        }
  
        try {
          navigator.mediaDevices.getUserMedia(constraints).then(stream => {
            ctx.getCamera(stream, video);
          });
        } catch (e) {
          alert("This Browser is not supported.")
          console.error(e)
        }
      },
      predictWebcam() {
        const ctx = this;
        try {
          const liveView = document.getElementById('liveView')
          this.model?.detect(this.video).then(function (predictions) {
            for (let i = 0; i < ctx.children.length; i++) {
            }
            ctx.children.splice(0);
            for (let n = 0; n < predictions.length; n++) {
              ctx.isInside = predictions[n].score > 0.66 && predictions[n].class === "car";
            }
            window.requestAnimationFrame(ctx.predictWebcam);
          });
        } catch (e) {
          ctx.err = e
        }
      },
      takePicture() {
        this.canvas = document.getElementById('canvas');
        this.photo = document.getElementById('photo');
        let context = this.canvas.getContext('2d');
        const vw = Math.max(document.documentElement.clientWidth || 0, window.innerWidth || 0)
        const vh65 = Math.ceil(Math.max(document.documentElement.clientHeight || 0, window.innerHeight || 0) * 0.65)
        this.canvas.width = vw;
        this.canvas.height = vh65;
        const video = document.querySelector('#video');
  
        try {
          context.drawImage(video, 0, 0, vw, vh65);
          const ctx = this;
          if (this.activeItem > 0) {
            let tag = ctx.stages[ctx.activeItem].tag;
            this.canvas.toBlob((blob) => {
              ctx.photos[tag].url = URL.createObjectURL(blob);
              ctx.photos[tag].blob = blob;
            }, 'image/jpeg');
            this.photo.setAttribute('src', this.photos[this.stages[tag]]);
            console.log(this.photos)
          }
        } catch (e) {
          this.err = e
        }
      },
    },
    watch: {
      interval(v) {
        if (v <= 0) {
          clearInterval(this.interval);
        }
      },
      time(val) {
        if ((val === 0)) {
          this.timeOut = true;
          clearInterval(this.interval);
          this.streaming = false;
  
          this.$bvModal.msgBoxConfirm("Vehicle Inspection Timed out, you will have to start all over again and make sure you don't exceed one minute time given", {
            title: 'Inspection Timeout',
            size: 'sm',
            buttonSize: 'sm',
            okVariant: 'dark',
            okTitle: 'Restart',
            centered: true,
            headerClass: "text-danger"
          }).then((e) => {
            this.resetProcess()
            if (e === true) {
              this.snap()
            }
          })
  
        }
      },
      photos(val) {
        if (val.length === 6) {
          if (document.fullscreenElement) {
            document.exitFullscreen().then(() => {
            }).catch((err) => {
              console.error(err)
            })
          } else {
            document.documentElement.requestFullscreen();
          }
        }
      },
      activeItem(e) {
        if (e === this.stages.length) this.endCapturing();
      }
    },
  }
  </script>
  
  <style scoped>
  .screen {
    height: 100vh;
    width: 100vw;
    overflow: hidden;
    position: absolute;
    top: 0;
    left: 0;
    -moz-user-select: none;
    -webkit-user-select: none;
    -ms-user-select: none;
    user-select: none;
    -o-user-select: none;
  }
  
  .font-weight-medium {
    font-weight: 500;
  }
  
  .text-green {
    color: var(--teal) !important;
  }
  
  .video-bg {
    background: linear-gradient(0deg, rgba(0, 0, 0, 0.9), rgba(0, 0, 0, 0.9));
    height: 65vh;
  }
  
  .video-bg video {
    height: 100%;
    width: 100vw;
  }
  
  
  .action-btn {
    width: 71px;
    height: 71px;
    background: #000000;
    border: 4px solid #f6f3f3;
    display: flex;
    border-radius: 100px;
    align-items: center;
    justify-content: center;
    position: absolute;
    top: -40px;
    z-index: 4;
    backdrop-filter: opacity(0);
    /*border-image: linear-gradient(to right, white, #FEFEFE);*/
  }
  
  .action-btn .rim {
    width: 48px;
    height: 48px;
    border: 2px solid #00FFBF;
    border-radius: 100px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 12px;
    font-weight: 600;
  }
  
  canvas {
    height: 65vh;
    width: 100vw;
  }
  
  #photo {
    -webkit-transform: scaleX(-1);
    transform: scaleX(-1);
  }
  
  .director {
    bottom: 95px;
    z-index: 20;
    position: absolute;
    width: 100%;
    left: 0;
  }
  
  .director img {
    width: 260px;
  }
  
  .counter {
    width: 50px;
    height: 50px;
    background-color: transparent;
    position: absolute;
    top: 30px;
    left: 20px;
    z-index: 21;
  }
  
  .counter div {
    width: 42px;
    height: 42px;
    /*color: #C60909;*/
  }
  
  .stages {
    width: 43px;
    height: max-content;
    /*background-color: transparent;*/
    position: absolute;
    padding: 10px 0 10px 0;
    top: 30px;
    right: 10px;
    z-index: 20;
    background: rgba(0, 0, 0, 0.6);
    border-radius: 100px;
  }
  
  .stage {
    width: 30px;
    height: 30px;
    background: rgba(255, 255, 255, 0.35);
  }
  
  .bg-green {
    background-color: #4FBFA3 !important;
  }
  
  .action-pane {
    height: 35vh;
  }
  
  .stage-image {
    width: 50vw;
    height: auto;
  }
  
  .loading {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    z-index: 2;
  }
  
  @media all and (display-mode: fullscreen) {
    .bg-white, .action-pane {
      background-color: white !important;
    }
  }
  
  .ai-video-container {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    z-index: 2;
    width: 90vw;
    height: 62vh;
    color: rgba(255, 255, 255, 0.78);
    border-radius: 10px;
  }
  
  .dark {
    /*border: 4px solid red;*/
    border: 4px solid #05eeb4;
  }
  
  .light {
    border: 4px solid #05eeb4;
  }
  
  :fullscreen {
    background-color: white !important;
  }
  
  .action-btn:disabled {
    opacity: 0.5;
  }
  
  .scrollable-yy {
    max-height: 500px;
    overflow-y: auto;
  }
  
  </style>
  