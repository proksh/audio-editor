<template>
  <AudioEditor
    audioTotalLengthInSeconds="10"
    toTimeInMilliSeconds="3"
    fromTimeInMilliSeconds="0"
    fadeInDurationInMilliSeconds="1000"
    fadeOutDurationInMilliSeconds="1000"
  />
</template>

<script>
import AudioEditor from "./components/AudioEditor.vue";

export default {
  name: "App",
  components: {
    AudioEditor,
  },
};
</script>

<style></style>
