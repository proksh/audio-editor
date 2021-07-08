<template>
  <div class="audio-editor">
    <div class="wrapper" ref="wrapper">
      <!-- Waveform fader at top for volume -->
      <div class="volume-top-fader" :style="volumeFaderStyle">
        <div
          class="volume-controller"
          :style="{ width: `${selectionWidth}px` }"
        >
          <div class="upper-stroke" :style="upperStrokeStyle" />
          <div ref="volumeDrag" class="volume-dragger" />
        </div>
      </div>
      <!-- Waveform fader at start -->
      <div class="fade-waveform left" :style="leftFadeStyle" />
      <!-- Start stroke and fader -->
      <div class="start" :style="matchHeight">
        <div class="stroke" :style="startStrokeStyle" />
        <div ref="startFade" :style="startFaderStyle" class="start-fader" />
      </div>
      <!-- Main selection area with waveform and trapezoid canvas -->
      <div
        class="selection-wrapper"
        :style="{ height: `${selectionHeight}px` }"
      >
        <div ref="selection" class="selection" :style="selectionStyle">
          <canvas ref="trapezoid" id="trapezoid" />
          <img
            :style="waveformStyle"
            src="/images/waveform.svg"
            ref="waveform"
            class="waveform"
          />
        </div>
      </div>
      <!-- End stroke and fader -->
      <div class="end" :style="matchHeight">
        <div class="stroke" :style="endStrokeStyle" />
        <div ref="endFade" class="end-fader" :style="endFaderStyle" />
      </div>
      <!-- Waveform fader at end -->
      <div class="fade-waveform right" :style="rightFadeStyle" />
    </div>
  </div>
</template>

<script>
import interact from "interactjs";

const fadeLimit = 200;

let limits = {
  dragX: { min: 0, max: 0 },
  volume: { min: 0, max: 1 },
  fadeX: { min: 0, max: 0 },
};

export default {
  name: "HelloWorld",
  data() {
    return {
      selectionWidth: 600,
      selectionHeight: 150,
      volume: 0.8,
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
    // Set max limit for drag event
    limits.dragX.max = this.$refs.waveform.clientWidth - this.selectionWidth;
    // Set limit for fade event
    limits.fadeX.max =
      fadeLimit *
      (this.selectionWidth /
        (this.toTimeInMilliSeconds - this.fromTimeInMilliSeconds));
  },
  methods: {
    /**
    Drag
    **/
    initInteract: function(selector) {
      interact(selector).draggable({
        inertia: true,
        onmove: this.dragMoveListener,
        cursorChecker: this.dragCursorChecker,
      });
    },
    dragCursorChecker: function() {
      if (this.dragX <= limits.dragX.min + 5) {
        return "url(/images/right-only.svg) 13 9,zoom-out";
      } else if (this.dragX >= limits.dragX.max - 5) {
        return "url(/images/left-only.svg) 13 9,zoom-out";
      } else {
        return "url(/images/grab.svg) 13 9,zoom-out";
      }
    },
    dragMoveListener: function(event) {
      let dragX = this.dragX + event.dx;
      if (dragX < limits.dragX.min) {
        dragX = limits.dragX.min;
      } else if (dragX > limits.dragX.max) {
        dragX = limits.dragX.max;
      }
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
      let startFadeX = parseFloat(this.startFadeX) + event.dx;
      if (startFadeX < limits.fadeX.min) {
        startFadeX = limits.fadeX.min;
      } else if (startFadeX > limits.fadeX.max) {
        startFadeX = limits.fadeX.max;
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
      let endFadeX = parseFloat(this.endFadeX) + event.dx;
      if (endFadeX < -limits.fadeX.max) {
        endFadeX = -limits.fadeX.max;
      } else if (endFadeX > limits.fadeX.min) {
        endFadeX = limits.fadeX.min;
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
      let volume =
        (parseFloat(this.selectionHeight) * parseFloat(this.volume) -
          event.dy) /
        parseFloat(this.selectionHeight);
      if (volume < 0) {
        volume = 0;
      } else if (volume > 1) {
        volume = 1;
      }
      this.volume = volume;
    },
    // Canvas
    updateCanvas: function() {
      var canvas = document.getElementById("trapezoid"),
        shape = canvas.getContext("2d");
      canvas.height = this.selectionHeight * this.volume;
      canvas.width = this.selectionWidth;
      shape.clearRect(
        0,
        0,
        this.selectionWidth,
        this.selectionHeight * this.volume
      );
      shape.fillStyle = "#e6ddf8";
      shape.beginPath();
      shape.moveTo(this.startFadeX, 0);
      shape.lineTo(this.selectionWidth + this.endFadeX, 0);
      shape.lineTo(this.selectionWidth, this.selectionHeight * this.volume);
      shape.lineTo(0, this.selectionHeight * this.volume);
      shape.closePath();
      shape.fill();
    },
  },
  watch: {
    dragX: function() {
      const selectionLength =
        this.toTimeInMilliSeconds - this.fromTimeInMilliSeconds;
      const startTime = (this.dragX * selectionLength) / this.selectionWidth;
      const endTime = startTime + selectionLength;
      console.log(startTime, endTime);
      // selectionChanged (startTime, endTime)
    },
    startFadeX: function() {
      this.updateCanvas();
      const selectionLength =
        this.toTimeInMilliSeconds - this.fromTimeInMilliSeconds;
      const duration =
        (this.startFadeX * selectionLength) / this.selectionWidth;
      console.log(duration);
      // fadeInChanged(duration);
    },
    endFadeX: function() {
      this.updateCanvas();
      const selectionLength =
        this.toTimeInMilliSeconds - this.fromTimeInMilliSeconds;
      const duration = (-this.endFadeX * selectionLength) / this.selectionWidth;
      console.log(duration);
      // fadeOutChanged(duration);
    },
    selectionHeight: function() {
      this.updateCanvas();
    },
    volume: function() {
      this.updateCanvas();
      console.log(this.volume);
      // volumeChanged(this.volume);
    },
    audioTotalLengthInSeconds: function() {
      limits.dragX.max =
        this.$refs.waveform.clientWidth - this.$refs.selection.clientWidth;
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
    volumeFaderStyle: function() {
      return {
        height: `${1 +
          (1 - parseFloat(this.volume)) * parseFloat(this.selectionHeight)}px`,
      };
    },
    leftFadeStyle: function() {
      return {
        height: `${this.selectionHeight * this.volume}px`,
        transform: `skewX(-${(180 *
          Math.atan(this.startFadeX / (this.selectionHeight * this.volume))) /
          Math.PI}deg)scaleX(2)`,
      };
    },
    startStrokeStyle: function() {
      return {
        transform: `skewX(-${(180 *
          Math.atan(this.startFadeX / (this.selectionHeight * this.volume))) /
          Math.PI}deg)`,
      };
    },
    startFaderStyle: function() {
      return { transform: `translateX(${this.startFadeX}px)` };
    },
    selectionStyle: function() {
      return {
        width: `${this.selectionWidth}px`,
        height: `${this.selectionHeight * this.volume}px`,
      };
    },
    waveformStyle: function() {
      return {
        transform: `translateX(${-this.dragX}px)`,
        width: `${(100000 * this.audioTotalLengthInSeconds) /
          (this.toTimeInMilliSeconds - this.fromTimeInMilliSeconds)}%`,
        height: `${this.selectionHeight}px`,
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
          Math.atan(-this.endFadeX / (this.selectionHeight * this.volume))) /
          Math.PI}deg)`,
      };
    },
    endFaderStyle: function() {
      return { transform: `translateX(${this.endFadeX}px)` };
    },
    rightFadeStyle: function() {
      return {
        height: `${this.selectionHeight * this.volume}px`,
        transform: `skewX(${(180 *
          Math.atan(-this.endFadeX / (this.selectionHeight * this.volume))) /
          Math.PI}deg)scaleX(2)`,
      };
    },
    matchHeight: function() {
      return { height: `${this.selectionHeight * this.volume}px` };
    },
  },
};
</script>

<style scoped>
.audio-editor {
  padding: 100px 0;
  background: #fafafa;
  overflow: hidden;
}
.wrapper {
  position: relative;
  display: flex;
  align-items: flex-end;
  justify-content: center;
  width: 100%;
  overflow: visible;
}
.volume-top-fader {
  position: absolute;
  top: -1px;
  left: 0;
  right: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-end;
  background: #fafafa;
  opacity: 0.8;
  z-index: 1;
}
.volume-controller {
  position: relative;
}
.volume-dragger {
  position: absolute;
  height: 16px;
  background: white;
  width: 30px;
  top: -10px;
  border: 2px solid black;
  left: 50%;
  border-radius: 10px;
  transform: translateX(-50%);
  cursor: row-resize !important;
}
.upper-stroke {
  height: 2px;
  background: black;
}
.selection-wrapper {
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
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
.waveform {
  position: absolute;
  bottom: 1px;
  left: 0;
  width: 250%;
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
  top: -16px;
}
.start-fader {
  left: -12px;
}
.end-fader {
  right: -14px;
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
