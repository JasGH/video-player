<template>
<v-card
    class="mx-auto"
    width="1000px"
  >
 <video-player :source="sources" :time-points="timePoints" :poster="poster"/>
</v-card>
</template>

<script>
  import VideoPlayer from "../components/VideoPlayer";
  export default {
    name: 'Home',
    props:{},
    components: {
      VideoPlayer,
    },
    data(){
      return {
        timePoints : [
          {
            title : 'تست ۱',
            time : 18
          },
          {
            title : 'تست ۲',
            time : 188
          },
          {
            title : 'تست ۳',
            time : 400
          },
          {
            title : 'تست ۴',
            time : 800
          },
        ],
        sources: [
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
        ],
        poster : 'https://cdn.alaatv.com/media/thumbnails/1042/1042002xxxx.jpg?w=114&h=64'
      }
    }
  }
</script>
