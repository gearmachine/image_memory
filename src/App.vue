<template>
  <v-app>
    <!-- <v-app-bar
      app
      color="primary"
      dark
    >
    </v-app-bar> -->
    <!-- <div ref="force"></div>-->
    <canvas ref="image">
    </canvas>
    <canvas ref="circle">
    </canvas>
    <canvas ref="canvas">
      Sorry, your browser is too old for this demo.
    </canvas>

    <div v-if="!show" style="width: 200px; z-index: 1; position: abusolute">
      <div ref="touches"></div>
      <v-file-input
        label="手本画像" 
        prepend-icon="mdi-image"
        v-model="inputImage" 
        accept="image/*" 
        outlined
        dense
        @change="onImagePicked"
      >
      </v-file-input>
      <v-text-field 
        outlined
        dense
        label='半径'
        v-model="radius">
      </v-text-field>
      <v-btn @click="show = !show">隠す {{debugHeight}}</v-btn>
    </div>

  </v-app>
</template>

<script>

export default {
  name: 'App',
  created() {
    const viewportContent = "initial-scale=1.0, maximum-scale=1.0, user-scalable=no";
    document.querySelector("meta[name='viewport']").setAttribute("content", viewportContent);
  },
  mounted() {
    // const $force = this.$refs.force;
    const $touches = this.$refs.touches;
    const canvas = this.$refs.canvas;
    if (!canvas || !canvas.getContext) return false;
    const context = canvas.getContext('2d')
    let lineWidth = 0
    let isMousedown = false
    let points = []

    this.initCircleCanvas(canvas);

    canvas.width = window.innerWidth * 2
    canvas.height = window.innerHeight * 2
    this.debugHeight = canvas.height;

    const requestIdleCallback = window.requestIdleCallback || function (fn) { setTimeout(fn, 1) };
    for (const ev of ["touchstart", "mousedown"]) {
      canvas.addEventListener(ev, function (e) {
        let pressure = 0.1;
        let x, y;
        if (e.touches && e.touches[0] && typeof e.touches[0]["force"] !== "undefined") {
          if (e.touches[0]["force"] > 0) {
            pressure = e.touches[0]["force"]
          }
          x = e.touches[0].pageX * 2
          y = e.touches[0].pageY * 2
        } else {
          pressure = 0.2
          x = e.pageX * 2
          y = e.pageY * 2
        }

        isMousedown = true

        lineWidth = Math.log(pressure + 1) * 40
        context.lineWidth = lineWidth// pressure * 50;
        context.strokeStyle = 'black'
        context.lineCap = 'round'
        context.lineJoin = 'round'
        context.beginPath()
        context.moveTo(x, y)

        points.push({ x, y, lineWidth })
      })
    }

    for (const ev of ['touchmove', 'mousemove']) {
      canvas.addEventListener(ev, function (e) {
        if (!isMousedown) return
        e.preventDefault()

        let pressure = 0.1
        let x, y
        if (e.touches && e.touches[0] && typeof e.touches[0]["force"] !== "undefined") {
          if (e.touches[0]["force"] > 0) {
            pressure = e.touches[0]["force"]
          }
          x = e.touches[0].pageX * 2
          y = e.touches[0].pageY * 2
        } else {
          pressure = 0.2
          x = e.pageX * 2
          y = e.pageY * 2
        }

        // smoothen line width
        lineWidth = (Math.log(pressure + 1) * 40 * 0.2 + lineWidth * 0.8)
        points.push({ x, y, lineWidth })

        context.strokeStyle = 'black'
        context.lineCap = 'round'
        context.lineJoin = 'round'
        // context.lineWidth   = lineWidth// pressure * 50;
        // context.lineTo(x, y);
        // context.moveTo(x, y);

        if (points.length >= 3) {
          const l = points.length - 1
          const xc = (points[l].x + points[l - 1].x) / 2
          const yc = (points[l].y + points[l - 1].y) / 2
          context.lineWidth = points[l - 1].lineWidth
          context.quadraticCurveTo(points[l - 1].x, points[l - 1].y, xc, yc)
          context.stroke()
          context.beginPath()
          context.moveTo(xc, yc)
        }

        requestIdleCallback(() => {
          // $force.textContent = 'force = ' + pressure

          const touch = e.touches ? e.touches[0] : null
          if (touch) {
            $touches.innerHTML = `x: ${Math.floor(x)}, y: ${Math.floor(y)}`;
            // $touches.innerHTML = `
            //   touchType = ${touch.touchType} ${touch.touchType === 'direct' ? '👆' : '✍️'} <br/>
            //   radiusX = ${touch.radiusX} <br/>
            //   radiusY = ${touch.radiusY} <br/>
            //   rotationAngle = ${touch.rotationAngle} <br/>
            //   altitudeAngle = ${touch.altitudeAngle} <br/>
            //   azimuthAngle = ${touch.azimuthAngle} <br/>
            // `

            // 'touchev = ' + (e.touches ? JSON.stringify(
            //   ['force', 'radiusX', 'radiusY', 'rotationAngle', 'altitudeAngle', 'azimuthAngle', 'touchType'].reduce((o, key) => {
            //     o[key] = e.touches[0][key]
            //     return o
            //   }, {})
            // , null, 2) : '')
          }
        })
      })
    }

    for (const ev of ['touchend', 'touchleave', 'mouseup']) {
      canvas.addEventListener(ev, function (e) {
        // let pressure = 0.1;
        let x, y;

        if (e.touches && e.touches[0] && typeof e.touches[0]["force"] !== "undefined") {
          if (e.touches[0]["force"] > 0) {
            // pressure = e.touches[0]["force"]
          }
          x = e.touches[0].pageX * 2
          y = e.touches[0].pageY * 2
        } else {
          // pressure = 1.0
          x = e.pageX * 2
          y = e.pageY * 2
        }

        isMousedown = false

        context.strokeStyle = 'black'
        context.lineCap = 'round'
        context.lineJoin = 'round'

        if (points.length >= 3) {
          const l = points.length - 1
          context.quadraticCurveTo(points[l].x, points[l].y, x, y)
          context.stroke()
        }

        points = []
        lineWidth = 0
      })
    }
    this.loadImage(this.uploadImageUrl);
  },
  data: () => ({
    radius: 250,
    show: false,
    inputImage: null,
    debugHeight: 0,
    uploadImageUrl: 'https://vignette.wikia.nocookie.net/splatoon/images/7/7f/SSBU_Inkling_Girl.png',
  }),
  methods: {
    initCircleCanvas(parentCanvas) {
      const canvas = this.$refs.circle;
      if (!canvas || !canvas.getContext) return false;
      const context = canvas.getContext('2d');
      let isMousedown = false;

      canvas.width = window.innerWidth * 2;
      canvas.height = window.innerHeight * 2;
      let radius = this.radius;

      for (const ev of ["touchstart", "mousedown"]) {
        parentCanvas.addEventListener(ev, function (e) {
          let x, y;

          if (e.touches && e.touches[0] && typeof e.touches[0]["force"] !== "undefined") {
            x = e.touches[0].pageX * 2;
            y = e.touches[0].pageY * 2;
          } else { 
            x = e.pageX * 2;
            y = e.pageY * 2;
          }
          isMousedown = true;
          context.clearRect(0, 0, canvas.width, canvas.height)
          context.beginPath();
          context.fillStyle = 'rgb(255, 255, 255)';
          context.arc(x, y, radius, 0, 2 * Math.PI);
          context.fill();
        })
      }

      for (const ev of ['touchmove', 'mousemove']) {
        parentCanvas.addEventListener(ev, function (e) {
          if (!isMousedown) return
          e.preventDefault()

          let x, y
          if (e.touches && e.touches[0] && typeof e.touches[0]["force"] !== "undefined") {
            x = e.touches[0].pageX * 2
            y = e.touches[0].pageY * 2
          } else {
            x = e.pageX * 2
            y = e.pageY * 2
          }
          context.clearRect(0, 0, canvas.width, canvas.height)
          context.beginPath();
          context.fillStyle = 'rgb(255, 255, 255)';
          context.arc(x, y, radius, 0, 2 * Math.PI);
          context.fill();

        })
      }

      for (const ev of ['touchend', 'touchleave', 'mouseup']) {
        parentCanvas.addEventListener(ev, function () {
          context.clearRect(0, 0, canvas.width, canvas.height)
          isMousedown = false
        })
      }
    },
    onImagePicked(file) {
      if (file !== undefined && file !== null) {
        if (file.name.lastIndexOf('.') <= 0) {
          return;
        }
        const fr = new FileReader()
        fr.readAsDataURL(file)
        fr.addEventListener('load', () => {
          this.uploadImageUrl = fr.result;
          this.loadImage(this.uploadImageUrl);
          this.fileLoaded = true;
        })
      } else {
        this.uploadImageUrl = '';
      }
    },
    loadImage(imageUrl) {
      const canvas = this.$refs.canvas;
      let canvasImage = this.$refs.image;
      canvasImage.width = canvas.width;
      canvasImage.height = canvas.height;
      if (!canvasImage || !canvasImage.getContext) return false;
      const contextImage = canvasImage.getContext('2d');
      const image = new Image();
      image.src = imageUrl;

      image.onload = () => {
        let width
        let height;

        // 横長度
        const imageAspect = image.width / image.height;
        const canvasAspect = parseFloat(canvasImage.width) / parseFloat(canvasImage.height);
        console.log("imageAspect", imageAspect);
        console.log("canvasAspect", canvasAspect);


        if (imageAspect < canvasAspect) {
          console.log("canvasAspect > imageAspect")
          width =  canvasImage.height * imageAspect;
          height = canvasImage.height;
        } else {
          console.log("!canvasAspect > imageAspect")
          width =  canvasImage.width;
          height = canvasImage.width / imageAspect;
        }
        contextImage.drawImage( image , 0 , 0 , width , height);
      };
    },
  },
};
</script>
<style>
  html, body {
    overflow: hidden;
  }
  body {
    position: fixed;
    margin: 0;
    left: 0;
    top: 0;
    width: 100vw;
    height: 100%;
    font-family: sans-serif;
    font-size: 13px;
    padding: 0px;
    box-sizing: border-box;
  }
  canvas {
    position: fixed;
    width: 100vw;
    height: 100%;
    left: 0;
    top: 0;
  }
</style>