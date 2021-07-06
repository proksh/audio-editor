<template>
  <div class="wrapper">
    <div class="fader" />
    <div class="start" />
    <div ref="selection" class="selection">
      <img
        v-bind:style="{
          width:
            (100 * audioTotalLengthInSeconds) /
              (toTimeInMilliSeconds - fromTimeInMilliSeconds) +
            '%',
        }"
        src="/images/waveform.svg"
        ref="waveform"
        class="waveform"
      />
    </div>
    <div class="end" />
    <div class="fader" />
  </div>
</template>

<script>
import interact from "interactjs";

export default {
  name: "HelloWorld",
  mounted() {
    let selection = this.$refs.selection;
    this.initInteract(selection);
  },
  methods: {
    initInteract: function(selector) {
      interact(selector).draggable({
        inertia: true,
        onmove: this.dragMoveListener,
        onend: this.onDragEnd,
      });
    },
    dragMoveListener: function(event) {
      const waveform = this.$refs.waveform;
      const target = event.target;
      const minX = 0;
      const maxX = waveform.clientWidth - target.clientWidth;
      let x = (parseFloat(target.getAttribute("data-x")) || 0) + event.dx;
      if (x < minX) {
        x = minX;
      } else if (x > maxX) {
        x = maxX;
      }

      waveform.style.webkitTransform = waveform.style.transform =
        "translateX(" + -x + "px)";

      target.setAttribute("data-x", x);
    },
    onDragEnd: function() {
      console.log("dragend");
    },
  },
  props: {
    audioTotalLengthInSeconds: Number,
    toTimeInMilliSeconds: Number,
    fromTimeInMilliSeconds: Number,
    fadeInDurationInMilliSeconds: Number,
    fadeOutDurationInMilliSeconds: Number,
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
  cursor: url(/images/grab.svg) 43 43, -webkit-zoom-out;
  cursor: url(/images/grab.svg) 43 43, -moz-zoom-out;
  cursor: url(/images/grab.svg) 43 43, zoom-out;
  cursor: grab !important;
  position: relative;
  background: #e6ddf8;
  height: 150px;
  width: 600px;
  border-bottom: 2px solid black;
}
.waveform {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 250%;
  height: 100px;
}
.start,
.end {
  min-width: 2px;
  height: 152px;
  background: black;
  position: relative;
  z-index: 2;
}
.fader {
  position: relative;
  z-index: 2;
  flex-grow: 1;
  background: #fafafa;
  opacity: 0.8;
  height: 400px;
}
</style>
