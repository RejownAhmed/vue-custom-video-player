<script setup>
import videoPlayer from './components/videoPlayer.vue'

</script>

<template>
  <div>
    <video-player
      url="/video.mp4"
      caption="/subtitles.vtt"
    >
    </video-player>
  </div>
</template>

<style scoped>

</style>
