<template>
  <div>
    <video-player :source="sources" />
  </div>
</template>

<script>
import VideoPlayer from "@/components/VideoPlayer.vue";


export default {
  name: "VideoExample",
  components: {
    VideoPlayer
  },
  data() {
    return {
      sources:  []
    }
  }
};
</script>