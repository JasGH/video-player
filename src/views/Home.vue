<template>
  <div class="video-js" >
    <video-player :options="videoOptions"/>
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
      videoOptions: {
        autoplay: true,
        controls: true,
        sources: [
          {
            src:
                "http://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ElephantsDream.mp4",
            type: "video/mp4"
          }
        ]
      }
    };
  }
};
</script>
