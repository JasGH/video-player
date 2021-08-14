<template>
  <div>
    <video ref="videoPlayer" class="video-js vjs-fluid vjs-big-play-centered"></video>
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
  props:{
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
          sources: []
        }
      };
    },
  created() {
    this.sources = this.source
  },
  mounted() {
    this.player = videojs(
        this.$refs.videoPlayer, this.options, function onPlayerReady() {
          console.log('onPlayerReady', this);
        })
  },
  beforeDestroy() {
    if (this.player) {
      this.player.dispose()
    }
  }
}
</script>