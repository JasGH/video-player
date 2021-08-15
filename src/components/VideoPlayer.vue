<template>
  <div>

    <v-btn
        class="vPlayer-drawer-btn"
        text
        icon
        x-large
        @click.stop="drawer = true"
    >
      <v-icon>mdi-menu</v-icon>
    </v-btn>
    <v-navigation-drawer
        right
        v-model="drawer"
        absolute
        bottom
        temporary
    >
      <v-list-item class="px-2">
        <v-list-item-title>زمانکوب ها</v-list-item-title>
        <v-btn
            icon
            @click.stop="drawer = false"
        >
          <v-icon>mdi-chevron-right</v-icon>
        </v-btn>
      </v-list-item>
      <v-divider></v-divider>
      <v-list
          nav
          dense
      >
        <v-list-item-group
            active-class="deep-purple--text text--accent-4"
        >
          <v-list-item
              v-for="(time,index) in timePoints"
              :key="index"
              @click="activate(time.time)"
          >
            <v-list-item-title>{{ time.title }}</v-list-item-title>
          </v-list-item>
        </v-list-item-group>
      </v-list>
    </v-navigation-drawer>
    <video ref="videoPlayer" class="video-js vjs-fluid vjs-big-play-centered " ></video>
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
    },
    timePoints : {
      type :Array,
      default (){
        return [
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
        ]
      }
    }
  },
    data() {
      return {
        drawer : false,
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
  },
  beforeDestroy() {
    if (this.player) {
      this.player.dispose()
    }
  },
  methods:{
    activate(time){
      this.player.currentTime(time)
    }
  }
}
</script>

<style>
.vPlayer-drawer-btn{
  position: absolute !important;
  right: 0 !important;
  z-index: 10 !important;
}
.theme--light.v-navigation-drawer {
  background-color: #FFFFFF;
  z-index: 11;
}
.vPlayer-drawer-btn-close{

}
</style>