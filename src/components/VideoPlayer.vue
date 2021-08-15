<template>
  <div>
    <video ref="videoPlayer" class="video-js vjs-fluid vjs-big-play-centered " ></video>
    <v-btn
        elevation="2"
    ></v-btn>
  </div>
</template>

<script>
import videojs from 'video.js';
import fa from 'video.js/dist/lang/fa.json';
// The following registers the plugin with `videojs`
require('@silvermine/videojs-quality-selector')(videojs);
require('@silvermine/videojs-quality-selector/dist/css/quality-selector.css')

export default {
  name: "VideoPlayer",
  props: {
    source : {
      type :Array,
      default (){
        return []
      }
    }
  },
    data() {
      return {
        options: {
          controlBar: {
            children: [
              'playToggle',
              'progressControl',
              'volumePanel',
              'qualitySelector',
              'fullscreenToggle',
            ],
          },
          language: 'fa',
          languages: {
            fa
          },
          autoplay: true,
          controls: true,
          playbackRates: [0.2, 0.5, 1, 1.5, 2,3,4],
          // plugins: {
          //   hlsQualitySelector: {`
          //     displayCurrentQuality: true
          //   }
          // },
          nativeControlsForTouch : true,
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
          ]
        }
      };
    },
  created() {
    this.sources = this.source
    console.log(this.sources)
  },
  mounted() {
    this.player = videojs(
        this.$refs.videoPlayer, this.options, function onPlayerReady() {
          console.log('onPlayerReady', this);
        })
    this.player.currentTime()
  },
  beforeDestroy() {
    if (this.player) {
      this.player.dispose()
    }
  }
}
</script>