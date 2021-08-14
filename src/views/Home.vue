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
      sources:  [
          {
            src:
                "https://cdn.alaatv.com/media/1042/HD_720p/1042003xxxx.mp4",
            type: 'video/mp4',
            label: '720p',
            selected: true,
          },
        {
          src:
              "https://cdn.alaatv.com/media/1042/hq/1042003xxxx.mp4",
                  type: 'video/mp4',
            label: '480P',
            selected: true,
        },
        {
          src:
              "https://cdn.alaatv.com/media/1042/240p/1042003xxxx.mp4",
                  type: 'video/mp4',
            label: '240p',
            selected: true,
        }
      ]
        }
  }
};
</script>