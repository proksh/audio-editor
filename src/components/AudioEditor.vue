<template>
  <div class="wrapper" ref="wrapper">
    <!-- Waveform fader at start -->
    <div class="fade-waveform left" :style="leftFadeStyle" />
    <!-- Start stroke and fader -->
    <div class="start" :style="matchHeight">
      <div class="stroke" :style="startStrokeStyle" />
      <div ref="startFade" :style="startFaderStyle" class="start-fader" />
    </div>
    <!-- Main selection area with waveform and trapezoid canvas -->
    <div ref="selection" class="selection" :style="selectionStyle">
      <canvas ref="trapezoid" id="trapezoid" />
      <img
        :style="waveformStyle"
        src="/images/waveform.svg"
        ref="waveform"
        class="waveform"
      />
      <div class="upper-stroke" :style="upperStrokeStyle" />
      <div ref="volumeDrag" class="volume-dragger" />
    </div>
    <!-- End stroke and fader -->
    <div class="end" :style="matchHeight">
      <div class="stroke" :style="endStrokeStyle" />
      <div ref="endFade" class="end-fader" :style="endFaderStyle" />
    </div>
    <!-- Waveform fader at end -->
    <div class="fade-waveform right" :style="rightFadeStyle" />
  </div>
</template>

<script>
import interact from "interactjs";

export default {
  name: "HelloWorld",
  data() {
    return {
      selectionWidth: 600,
      selectionHeight: 150,
      dragX: 0,
      startFadeX: 0,
      endFadeX: 0,
    };
  },
  mounted() {
    this.initInteract(this.$refs.wrapper);
    this.initStartFadeInteract(this.$refs.startFade);
    this.initEndFadeInteract(this.$refs.endFade);
    this.initVolumeDragInteract(this.$refs.volumeDrag);
    this.updateCanvas();
    this.$refs.wrapper.setAttribute("data-dragX", this.dragX);
  },
  methods: {
    /**
    Drag
    **/
    initInteract: function(selector) {
      interact(selector).draggable({
        inertia: true,
        onmove: this.dragMoveListener,
        cursorChecker(action, interactable, element) {
          const dragX = parseFloat(element.getAttribute("data-dragX"));
          const maxX = parseFloat(element.getAttribute("data-maxX"));

          if (dragX <= 1) {
            return "url(/images/right-only.svg) 13 9,zoom-out";
          } else if (dragX >= maxX - 1) {
            return "url(/images/left-only.svg) 13 9,zoom-out";
          } else {
            return "url(/images/grab.svg) 13 9,zoom-out";
          }
        },
      });
    },
    dragMoveListener: function(event) {
      const target = event.target;
      const selection = this.$refs.selection;
      const waveform = this.$refs.waveform;
      const minX = 0;
      const maxX = waveform.clientWidth - selection.clientWidth;
      let dragX = this.dragX + event.dx;
      if (dragX < minX) {
        dragX = minX;
      } else if (dragX > maxX) {
        dragX = maxX;
      }
      target.setAttribute("data-dragX", dragX);
      target.setAttribute("data-maxX", maxX);
      this.dragX = dragX;
    },
    /**
    Start fade
    **/
    initStartFadeInteract: function(selector) {
      interact(selector).draggable({
        inertia: true,
        onmove: this.onFadeStartMove,
      });
    },
    onFadeStartMove: function(event) {
      const minStartFade = 0;
      const maxStartFace = 50;
      let startFadeX = parseFloat(this.startFadeX) + event.dx;
      if (startFadeX < minStartFade) {
        startFadeX = minStartFade;
      } else if (startFadeX > maxStartFace) {
        startFadeX = maxStartFace;
      }
      this.startFadeX = startFadeX;
    },
    /**
    End fade
    **/
    initEndFadeInteract: function(selector) {
      interact(selector).draggable({
        inertia: true,
        onmove: this.onFadeEndMove,
      });
    },
    onFadeEndMove: function(event) {
      const minEndFade = -50;
      const maxEndFace = 0;
      let endFadeX = parseFloat(this.endFadeX) + event.dx;
      if (endFadeX < minEndFade) {
        endFadeX = minEndFade;
      } else if (endFadeX > maxEndFace) {
        endFadeX = maxEndFace;
      }
      this.endFadeX = endFadeX;
    },
    /**
    Volumne Drag
    **/
    initVolumeDragInteract: function(selector) {
      interact(selector).draggable({
        inertia: true,
        onmove: this.onVolumeMove,
        cursorChecker() {
          return "row-resize";
        },
      });
    },
    onVolumeMove: function(event) {
      let selectionHeight = parseFloat(this.selectionHeight) - event.dy;
      if (selectionHeight < 100) {
        selectionHeight = 100;
      } else if (selectionHeight > 200) {
        selectionHeight = 200;
      }
      this.selectionHeight = selectionHeight;
    },
    // Canvas
    updateCanvas: function() {
      var canvas = document.getElementById("trapezoid"),
        shape = canvas.getContext("2d");

      canvas.height = this.selectionHeight;
      canvas.width = this.selectionWidth;

      shape.clearRect(0, 0, this.selectionWidth, this.selectionHeight);
      shape.fillStyle = "#e6ddf8";
      shape.beginPath();
      shape.moveTo(this.startFadeX, 0);
      shape.lineTo(this.selectionWidth + this.endFadeX, 0);
      shape.lineTo(this.selectionWidth, this.selectionHeight);
      shape.lineTo(0, this.selectionHeight);
      shape.closePath();
      shape.fill();
    },
  },
  watch: {
    startFadeX: function() {
      this.updateCanvas();
    },
    endFadeX: function() {
      this.updateCanvas();
    },
    selectionHeight: function() {
      this.updateCanvas();
    },
  },
  props: {
    audioTotalLengthInSeconds: Number,
    toTimeInMilliSeconds: Number,
    fromTimeInMilliSeconds: Number,
    fadeInDurationInMilliSeconds: Number,
    fadeOutDurationInMilliSeconds: Number,
  },
  computed: {
    leftFadeStyle: function() {
      return {
        height: `${this.selectionHeight}px`,
        transform: `skewX(-${(180 *
          Math.atan(this.startFadeX / this.selectionHeight)) /
          Math.PI}deg)scaleX(2)`,
      };
    },
    startStrokeStyle: function() {
      return {
        transform: `skewX(-${(180 *
          Math.atan(this.startFadeX / this.selectionHeight)) /
          Math.PI}deg)`,
      };
    },
    startFaderStyle: function() {
      return { transform: `translateX(${this.startFadeX}px)` };
    },
    selectionStyle: function() {
      return {
        width: `${this.selectionWidth}px`,
        height: `${this.selectionHeight}px`,
      };
    },
    waveformStyle: function() {
      return {
        transform: `scaleY(${this.selectionHeight / 150})translateX(${-this
          .dragX}px)`,
        width: `${(100 * this.audioTotalLengthInSeconds) /
          (this.toTimeInMilliSeconds - this.fromTimeInMilliSeconds)}%`,
      };
    },
    upperStrokeStyle: function() {
      return {
        transform: `translateX(${this.startFadeX}px)`,
        width: `${this.selectionWidth + this.endFadeX - this.startFadeX}px`,
      };
    },
    endStrokeStyle: function() {
      return {
        transform: `skewX(${(180 *
          Math.atan(-this.endFadeX / this.selectionHeight)) /
          Math.PI}deg)`,
      };
    },
    endFaderStyle: function() {
      return { transform: `translateX(${this.endFadeX}px)` };
    },
    rightFadeStyle: function() {
      return {
        height: `${this.selectionHeight}px`,
        transform: `skewX(${(180 *
          Math.atan(-this.endFadeX / this.selectionHeight)) /
          Math.PI}deg)scaleX(2)`,
      };
    },
    matchHeight: function() {
      return { height: `${this.selectionHeight}px` };
    },
  },
};
</script>

<style scoped>
.wrapper {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 400px;
  overflow: hidden;
  background: #fafafa;
}
.selection {
  position: relative;
}
.selection::after {
  content: "";
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 2px;
  background: black;
}
.upper-stroke {
  position: absolute;
  top: 0;
  left: 0;
  height: 2px;
  background: black;
}
.waveform {
  position: absolute;
  bottom: 1px;
  left: 0;
  width: 250%;
  height: 100px;
  transform-origin: bottom;
}
.start,
.end {
  position: relative;
  z-index: 3;
}
.start .stroke {
  position: absolute;
  top: 0;
  left: 0;
  bottom: 0;
  width: 2px;
  background: black;
  transform-origin: bottom left;
}
.end .stroke {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  width: 2px;
  background: black;
  transform-origin: bottom left;
}
.start-fader,
.end-fader {
  width: 24px;
  height: 24px;
  background: #fff;
  border-radius: 100%;
  border: 2px solid #000;
  position: absolute;
  top: -12px;
}
.start-fader {
  left: -12px;
}
.end-fader {
  right: -14px;
}
.volume-dragger {
  position: absolute;
  height: 16px;
  background: white;
  width: 30px;
  top: -8px;
  border: 2px solid black;
  left: 50%;
  border-radius: 10px;
  transform: translateX(-50%);
  cursor: row-resize !important;
}
.fade-waveform {
  position: relative;
  z-index: 2;
  flex-grow: 1;
  background: #fafafa;
  opacity: 0.8;
}
.fade-waveform.left {
  transform-origin: bottom right;
}
.fade-waveform.right {
  transform-origin: bottom left;
}
#trapezoid {
  position: absolute;
  top: 0;
  left: 0;
  z-index: 0;
}
</style>
