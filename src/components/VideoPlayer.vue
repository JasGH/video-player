<template>
    <div class="vPlayer" >
        <v-btn
            v-if="timePoints[0]"
            color="rgba(0, 0, 0, 0.6)"
            class="white--text vPlayer-drawer-btn"
            @click.stop="drawer = true"
        >
          <span class='vPlayer-timestamp-icon'></span>
        </v-btn>
        <div>
        <v-navigation-drawer
            right
            v-model="drawer"
            absolute
            :height="calcTheHeight"
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
                    active-class="indigo lighten-5 text--accent-4"
                >
                    <v-list-item
                        v-for="(timeStamp,index) in timePoints"
                        :key="index"
                        @click="activate(timeStamp.time)"
                    >
                        <v-list-item-action>
                            <v-list-item-action-text>
                                <v-menu bottom left>
                                    <template v-slot:activator="{ on }">
                                        <v-btn
                                            class="video-box-icon-button"
                                            icon
                                            @click.stop="toggleFavorite(timeStamp.id)"
                                            :loading="timeStamp.loading"
                                        >
                                            <i
                                                class="fi fi-rr-bookmark icon bookmark-button"
                                                :class="{ 'is-favorite': timeStamp.isFavored , 'is-not-favorite': !timeStamp.isFavored }"
                                            />
                                        </v-btn>
                                    </template>
                                </v-menu>
                            </v-list-item-action-text>
                        </v-list-item-action>
                        <v-list-item-title>{{ timeStamp.title }}</v-list-item-title>
                    </v-list-item>
                </v-list-item-group>
            </v-list>
        </v-navigation-drawer>
        </div>
        <video id="my-video" ref="videoPlayer" class="video-js vjs-fluid vjs-big-play-centered vjs-show-big-play-button-on-pause" data-setup="{}"></video>
    </div>
</template>

<script>
import videojs from 'video.js';
import fa from 'video.js/dist/lang/fa.json';
require("video.js/dist/video-js.css");
require('@silvermine/videojs-quality-selector')(videojs);
require('@silvermine/videojs-quality-selector/dist/css/quality-selector.css')
import hotkeys from 'videojs-hotkeys'
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
                return []
            }
        },
        poster : {
            type : String,
            default (){
                return ''
            }
        },
        keepCalculating : {
            type : Boolean,
            default (){
                return true
            }
        }
    },
    data() {
        return {
            drawer : false,
            favLoading : false,
            options: {
                controlBar: {
                    // currentTimeDisplay: true,
                    TimeDivider: true,
                    children: [
                        'playToggle',
                        'PlaybackRateMenuButton',
                        'CurrentTimeDisplay',
                        'progressControl',
                        'TimeDivider',
                        'RemainingTimeDisplay',
                        'volumePanel',
                        'SubtitlesButton',
                        'qualitySelector',
                        'fullscreenToggle',
                        'PictureInPictureToggle'
                    ],
                    volumePanel : {
                        inline: false,
                        vertical: true
                    },
                },
                language: 'fa',
                languages: {
                    fa
                },
                autoplay: false,
                controls: true,
                playbackRates: [0.25, 0.5, 1, 1.5, 2, 2.5, 3, 4],
                nativeControlsForTouch : true,
                sources: [],
                poster : null,
                plugins: {
                    hotkeys: {
                        enableModifiersForNumbers: false,
                        seekStep: 5,
                        enableMute : true,
                        enableVolumeScroll : true,
                        enableHoverScroll :true,
                        enableFullscreen :true
                    }
                }
            }
        };
    },
    created() {
        this.setSources()
        this.setPoster()
    },
    mounted() {
        let that = this
        this.player = videojs(
            this.$refs.videoPlayer, this.options, function onPlayerReady() {
                this.on('timeupdate', function () {
                if(that.keepCalculating){
                    that.calcWatchedPercentage(this.currentTime(),this.duration())
                }
                })
                document.querySelector('.video-js').focus()
            })
    },
    computed:{
        calcTheHeight(){
            return '100%'
        }
    },
    beforeDestroy() {
        if (this.player) {
            this.player.dispose()
        }
    },
    methods:{
        activate(time){
            this.player.currentTime(time)
            this.player.play()
            var requiredElement = document.querySelector('.video-js')
            requiredElement.focus()
        },
        setSources(){
            this.options.sources = this.source
        },
        setPoster(){
            this.options.poster = this.poster
        },
        reInitVideo(){
            this.player.src(this.source)
            this.player.poster(this.poster)
        },
        toggleFavorite(id){
            let that = this
            this.timePoints.forEach( function (item) {
                if (parseInt(item.id) === parseInt(id)) {
                    item.loading = true
                    item.isFavored = !!!item.isFavored
                    that.postIsFavored({
                        'id' : item.id,
                        'isFavored' : item.isFavored
                    } )
                }
            })
            var requiredElement = document.querySelector('.video-js')
            requiredElement.focus()
        },
        postIsFavored(timeStampData){
            var postStatus = 'unfavored'
            if (timeStampData.isFavored){
                postStatus = 'favored'
            }
            axios.post('/api/v2/c/timepoint/' + parseInt(timeStampData.id) + '/'+ postStatus)
                .then(response => {
                    if (response.status === 200){
                        this.timePoints.forEach( function (item) {
                            if (parseInt(item.id) === parseInt(timeStampData.id)) {
                                item.loading = false
                            }
                        })
                    }
                })
                .catch(err => console.error(err));
        },
        calcWatchedPercentage(currentTime , duration){
            const watchedPercentage = ((currentTime/duration) * 100)
            const videoPlayerTimeData = {
                'currentTime' : currentTime,
                'duration' : duration,
                'watchedPercentage' : watchedPercentage
            }
            this.$emit('calcTimeData' , videoPlayerTimeData)
        }
    },
    watch:{
        'source' : function (){
            this.reInitVideo()
        }
    }
}
</script>
// ---------------------------------------------------- all video js styles --------------------------------------
<style lang="scss"> // all of the skin
//.controls, .controls > * {
//  padding:0;
//  margin:0;
//}
//.controls {
//  overflow:hidden;
//  background:transparent;
//  width:100%;
//  height:8.0971659919028340080971659919028%; /* of figure's height */
//  position:relative;
//}
//.controls[data-state=hidden] {
//  display:none;
//}
//.controls[data-state=visible] {
//  display:block;
//}
//.controls > * {
//  float:left;
//  width:3.90625%;
//  height:100%;
//  margin-left:0.1953125%;
//  display:block;
//}
//.controls > *:first-child {
//  margin-left:0;
//}
//.controls .progress {
//  cursor:pointer;
//  width:75.390625%;
//}
//.controls button {
//  text-align:center;
//  overflow:hidden;
//  white-space:nowrap;
//  text-overflow:ellipsis;
//  border:none;
//  cursor:pointer;
//  text-indent:-99999px;
//  background:transparent;
//  background-size:contain;
//  background-repeat:no-repeat;
//}
//.controls button:hover, .controls button:focus {
//  opacity:0.5;
//}
//.controls button[data-state="play"] {
//  background-image: url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACgAAAAoCAYAAACM/rhtAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAA2ZpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuMy1jMDExIDY2LjE0NTY2MSwgMjAxMi8wMi8wNi0xNDo1NjoyNyAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wTU09Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9tbS8iIHhtbG5zOnN0UmVmPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvc1R5cGUvUmVzb3VyY2VSZWYjIiB4bWxuczp4bXA9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC8iIHhtcE1NOk9yaWdpbmFsRG9jdW1lbnRJRD0ieG1wLmRpZDpFNDZDNDg2MEEzMjFFMjExOTBEQkQ4OEMzRUMyQjhERCIgeG1wTU06RG9jdW1lbnRJRD0ieG1wLmRpZDpCNkU0NTY5NkE0MDcxMUUyQjgwQkYzQzhCMDZBRTU1NCIgeG1wTU06SW5zdGFuY2VJRD0ieG1wLmlpZDpCNkU0NTY5NUE0MDcxMUUyQjgwQkYzQzhCMDZBRTU1NCIgeG1wOkNyZWF0b3JUb29sPSJBZG9iZSBQaG90b3Nob3AgQ1M2IChXaW5kb3dzKSI+IDx4bXBNTTpEZXJpdmVkRnJvbSBzdFJlZjppbnN0YW5jZUlEPSJ4bXAuaWlkOjQzQ0QwNDBBMDJBNEUyMTFCOTZEQzYyRDgyRUVBOUZDIiBzdFJlZjpkb2N1bWVudElEPSJ4bXAuZGlkOkU0NkM0ODYwQTMyMUUyMTE5MERCRDg4QzNFQzJCOEREIi8+IDwvcmRmOkRlc2NyaXB0aW9uPiA8L3JkZjpSREY+IDwveDp4bXBtZXRhPiA8P3hwYWNrZXQgZW5kPSJyIj8+kBUJ9AAAAXFJREFUeNrsmLtOAkEUhneQyiAdDTExGlYMBaW9oq/ge8jlUbwkthTY2EGBLehbKK0UxsQgVK7/SWbMZo3j3mbmxPAnXyi2+fIzZ3dmRBAEHucUPO6hBhUyNXAH3umxJRZgCBo/nCKCe+DVoliUN5LUCd46lFOMwk4iPCRCiDl+Ko5X3RJOm99OEcGAyVyIrFO8lEPE9jXTBNvgRq4ba6+ZuAs5nFMwy3NQdFOcRpBSBtfgk6ugykkebZoUpGyBqyxtmhZUaYFnzoKqzcukbdoUVDkGT5wFKSVwEadNV4IqR3+16VrQkxuSVRxBVzvqKija+tQl/fafyx00u7/YBxOOU0yttcEHx9fMPphy/JJQa50krdkUrIMHjruZDdBN25ppwYOsrZkSpNZ68hDFast/Bg7Bo4nDu+7g/m/Oxc6u3+YMnBY6wTEDwXvdbmYXvDi82aKrP183xZQd0LcsSktrIC9PvV+neH1HvRZ0kC8BBgADq2RhyZa7BQAAAABJRU5ErkJggg==');
//}
//.controls button[data-state="pause"] {
//  background-image: url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACgAAAAoCAYAAACM/rhtAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAA2ZpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuMy1jMDExIDY2LjE0NTY2MSwgMjAxMi8wMi8wNi0xNDo1NjoyNyAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wTU09Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9tbS8iIHhtbG5zOnN0UmVmPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvc1R5cGUvUmVzb3VyY2VSZWYjIiB4bWxuczp4bXA9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC8iIHhtcE1NOk9yaWdpbmFsRG9jdW1lbnRJRD0ieG1wLmRpZDpFNDZDNDg2MEEzMjFFMjExOTBEQkQ4OEMzRUMyQjhERCIgeG1wTU06RG9jdW1lbnRJRD0ieG1wLmRpZDpCNzE0QzJGQUE0MDcxMUUyQjgwQkYzQzhCMDZBRTU1NCIgeG1wTU06SW5zdGFuY2VJRD0ieG1wLmlpZDpCNzAxODM5QUE0MDcxMUUyQjgwQkYzQzhCMDZBRTU1NCIgeG1wOkNyZWF0b3JUb29sPSJBZG9iZSBQaG90b3Nob3AgQ1M2IChXaW5kb3dzKSI+IDx4bXBNTTpEZXJpdmVkRnJvbSBzdFJlZjppbnN0YW5jZUlEPSJ4bXAuaWlkOjQzQ0QwNDBBMDJBNEUyMTFCOTZEQzYyRDgyRUVBOUZDIiBzdFJlZjpkb2N1bWVudElEPSJ4bXAuZGlkOkU0NkM0ODYwQTMyMUUyMTE5MERCRDg4QzNFQzJCOEREIi8+IDwvcmRmOkRlc2NyaXB0aW9uPiA8L3JkZjpSREY+IDwveDp4bXBtZXRhPiA8P3hwYWNrZXQgZW5kPSJyIj8+r7sqzQAAANdJREFUeNrs2MEKwjAMBuDGswd9C/UdPHvy6Ft6UTyKr6RDcceawDpKHZsE2kb4Az87GOiHNLCFvPfOcs2c9ZJ/MKSrDefCaeXnQmm7M9dfpgQoDY+CsDRy9moMeKqICznGJoqHhIie/JhXvnUNmxa9KQF6I3NBfzPFANYC7uTKRtkqeyZLOyQ0dLcVPRgSAAEEEEAAAQQQwJ9ftzQ92YAHzjLKXtmT7YUVX3UA5gK+DJiaMeDNAPCaToyl9dvdTazfpMIC810QJmed3cACk7CjBrByfQQYAHwMIXlfZRgfAAAAAElFTkSuQmCC');
//}
//.controls button[data-state="stop"] {
//  background-image: url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACgAAAAoCAYAAACM/rhtAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAA2ZpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuMy1jMDExIDY2LjE0NTY2MSwgMjAxMi8wMi8wNi0xNDo1NjoyNyAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wTU09Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9tbS8iIHhtbG5zOnN0UmVmPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvc1R5cGUvUmVzb3VyY2VSZWYjIiB4bWxuczp4bXA9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC8iIHhtcE1NOk9yaWdpbmFsRG9jdW1lbnRJRD0ieG1wLmRpZDpFNDZDNDg2MEEzMjFFMjExOTBEQkQ4OEMzRUMyQjhERCIgeG1wTU06RG9jdW1lbnRJRD0ieG1wLmRpZDpCNzAxODM5M0E0MDcxMUUyQjgwQkYzQzhCMDZBRTU1NCIgeG1wTU06SW5zdGFuY2VJRD0ieG1wLmlpZDpCNzAxODM5MkE0MDcxMUUyQjgwQkYzQzhCMDZBRTU1NCIgeG1wOkNyZWF0b3JUb29sPSJBZG9iZSBQaG90b3Nob3AgQ1M2IChXaW5kb3dzKSI+IDx4bXBNTTpEZXJpdmVkRnJvbSBzdFJlZjppbnN0YW5jZUlEPSJ4bXAuaWlkOjQzQ0QwNDBBMDJBNEUyMTFCOTZEQzYyRDgyRUVBOUZDIiBzdFJlZjpkb2N1bWVudElEPSJ4bXAuZGlkOkU0NkM0ODYwQTMyMUUyMTE5MERCRDg4QzNFQzJCOEREIi8+IDwvcmRmOkRlc2NyaXB0aW9uPiA8L3JkZjpSREY+IDwveDp4bXBtZXRhPiA8P3hwYWNrZXQgZW5kPSJyIj8+KkF/7gAAAOFJREFUeNrsmMsKwjAQRTNdu/ELpAUR/AVdC/6nu66kK/0hFXyBj22cQCohQqDUJFO4A4cusphDmFvaIa21klyFkl7mBltszZgt8zTHiXgzDTP/cfIEp8wtoZjP3UiGBOuMci2N60RuSIjoyo9x5ql7sdPo6+QJaiG5oMGkGIKpBddmVHuy7NKwa0gK+yronYNYIdGYQQhCEIIQhCAEIThoQZIuuPpDz0XMD1b81SHFsQUvApweIcGdAMG9nxh3u1UyJ5Vvs3VmqtD6zdSE2TCHhGJH27P0L42wo4Zg5voIMAB0bCBXvSa7VQAAAABJRU5ErkJggg==');
//}
//.controls button[data-state="mute"] {
//  background-image: url('https://i.ibb.co/PYftzKz/setting.png');
//}
//.controls button[data-state="unmute"] {
//  background-image: url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACgAAAAoCAYAAACM/rhtAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAA2ZpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuMy1jMDExIDY2LjE0NTY2MSwgMjAxMi8wMi8wNi0xNDo1NjoyNyAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wTU09Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9tbS8iIHhtbG5zOnN0UmVmPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvc1R5cGUvUmVzb3VyY2VSZWYjIiB4bWxuczp4bXA9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC8iIHhtcE1NOk9yaWdpbmFsRG9jdW1lbnRJRD0ieG1wLmRpZDpFNDZDNDg2MEEzMjFFMjExOTBEQkQ4OEMzRUMyQjhERCIgeG1wTU06RG9jdW1lbnRJRD0ieG1wLmRpZDpBMzk2MTA2OUE0MDcxMUUyQjgwQkYzQzhCMDZBRTU1NCIgeG1wTU06SW5zdGFuY2VJRD0ieG1wLmlpZDpBMzk2MTA2OEE0MDcxMUUyQjgwQkYzQzhCMDZBRTU1NCIgeG1wOkNyZWF0b3JUb29sPSJBZG9iZSBQaG90b3Nob3AgQ1M2IChXaW5kb3dzKSI+IDx4bXBNTTpEZXJpdmVkRnJvbSBzdFJlZjppbnN0YW5jZUlEPSJ4bXAuaWlkOjQzQ0QwNDBBMDJBNEUyMTFCOTZEQzYyRDgyRUVBOUZDIiBzdFJlZjpkb2N1bWVudElEPSJ4bXAuZGlkOkU0NkM0ODYwQTMyMUUyMTE5MERCRDg4QzNFQzJCOEREIi8+IDwvcmRmOkRlc2NyaXB0aW9uPiA8L3JkZjpSREY+IDwveDp4bXBtZXRhPiA8P3hwYWNrZXQgZW5kPSJyIj8+LD0czAAAARZJREFUeNrsly8LAkEQxW/lksUgGMxWs8VgEdRgEMwmv4Lfw2TwawgaxGI2WQ0Gm0WDBv+db3SFK4K3nnMjzIMfF/exzHs7Z4Ig8CQr5QmXGlSDf2fQGONKz8kh1UwUHNUBt6hnPc5jMNgEl+ddyDNYAQcy52rQRD2U5ulDFcEcZELjZKSkuACmYXOSaiYPxiAnsQfpxib2BsUVdRqM7Owl+pL0wemVzBCU1nLc8+KS4jM+vuOjwJJi32OUbjNqUA3+wOBVusGB3e9YFOe6RU/dDJSSLup3OoIaWEoOyQ40wFpyijegCraSa2YF6mAvuQcXoGVXM7FFTaluf9WdTD/uXdcfd67dbgiyLEWty4IaVIPCdBdgAJkkaR2v57S0AAAAAElFTkSuQmCC');
//}
//.controls button[data-state="volup"] {
//  background-image: url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACgAAAAoCAYAAACM/rhtAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAA2ZpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuMy1jMDExIDY2LjE0NTY2MSwgMjAxMi8wMi8wNi0xNDo1NjoyNyAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wTU09Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9tbS8iIHhtbG5zOnN0UmVmPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvc1R5cGUvUmVzb3VyY2VSZWYjIiB4bWxuczp4bXA9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC8iIHhtcE1NOk9yaWdpbmFsRG9jdW1lbnRJRD0ieG1wLmRpZDpFNDZDNDg2MEEzMjFFMjExOTBEQkQ4OEMzRUMyQjhERCIgeG1wTU06RG9jdW1lbnRJRD0ieG1wLmRpZDpCNzU1OEJFREE0MDcxMUUyQjgwQkYzQzhCMDZBRTU1NCIgeG1wTU06SW5zdGFuY2VJRD0ieG1wLmlpZDpCNzU1OEJFQ0E0MDcxMUUyQjgwQkYzQzhCMDZBRTU1NCIgeG1wOkNyZWF0b3JUb29sPSJBZG9iZSBQaG90b3Nob3AgQ1M2IChXaW5kb3dzKSI+IDx4bXBNTTpEZXJpdmVkRnJvbSBzdFJlZjppbnN0YW5jZUlEPSJ4bXAuaWlkOjQzQ0QwNDBBMDJBNEUyMTFCOTZEQzYyRDgyRUVBOUZDIiBzdFJlZjpkb2N1bWVudElEPSJ4bXAuZGlkOkU0NkM0ODYwQTMyMUUyMTE5MERCRDg4QzNFQzJCOEREIi8+IDwvcmRmOkRlc2NyaXB0aW9uPiA8L3JkZjpSREY+IDwveDp4bXBtZXRhPiA8P3hwYWNrZXQgZW5kPSJyIj8+1Pk2GwAAAQ9JREFUeNrsmL0KwjAQgHMOiogIPoLSQR9CcdDRxclXLIrg2mdSsYK41HjBVEJsM6XNld7BR6GF9mu4y8+BlFJQjo6gHmoEc3REyAG5qcc1cUViZPrnZAnOahazuSATl+A5oFxObDqBWSQA8MRLP3DWpeg0+jlZgpJIXUBjqrg1gmuVIRY7Hy/2lYNQ8vMZ5Rx8cw6yYNsENwVTCZRUsFndOQNkrNlXMc2A54HJKC91Peo52PUt+KBeJENk4fG7SwprsdA7ZN4PsiALVrny+BJcFdzbUtry87GzMYIvAk6pSzAhIJjYCWl2t+YibPtN9QkjV/tNheoRnvRQ1yV2R47i2zwVpe03rmIWDBAfAQYAByYx7rBsQ/AAAAAASUVORK5CYII=');
//}
//.controls button[data-state="voldown"] {
//  background-image: url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACgAAAAoCAYAAACM/rhtAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAA2ZpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuMy1jMDExIDY2LjE0NTY2MSwgMjAxMi8wMi8wNi0xNDo1NjoyNyAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wTU09Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9tbS8iIHhtbG5zOnN0UmVmPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvc1R5cGUvUmVzb3VyY2VSZWYjIiB4bWxuczp4bXA9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC8iIHhtcE1NOk9yaWdpbmFsRG9jdW1lbnRJRD0ieG1wLmRpZDpFNDZDNDg2MEEzMjFFMjExOTBEQkQ4OEMzRUMyQjhERCIgeG1wTU06RG9jdW1lbnRJRD0ieG1wLmRpZDpCNzU1OEJGMUE0MDcxMUUyQjgwQkYzQzhCMDZBRTU1NCIgeG1wTU06SW5zdGFuY2VJRD0ieG1wLmlpZDpCNzU1OEJGMEE0MDcxMUUyQjgwQkYzQzhCMDZBRTU1NCIgeG1wOkNyZWF0b3JUb29sPSJBZG9iZSBQaG90b3Nob3AgQ1M2IChXaW5kb3dzKSI+IDx4bXBNTTpEZXJpdmVkRnJvbSBzdFJlZjppbnN0YW5jZUlEPSJ4bXAuaWlkOjQzQ0QwNDBBMDJBNEUyMTFCOTZEQzYyRDgyRUVBOUZDIiBzdFJlZjpkb2N1bWVudElEPSJ4bXAuZGlkOkU0NkM0ODYwQTMyMUUyMTE5MERCRDg4QzNFQzJCOEREIi8+IDwvcmRmOkRlc2NyaXB0aW9uPiA8L3JkZjpSREY+IDwveDp4bXBtZXRhPiA8P3hwYWNrZXQgZW5kPSJyIj8+B37OGAAAANpJREFUeNrs2MEKwjAMBuDGk0efwSGCr6FXfU/FHeZV30hQ50U8dSmsMiLUCV0b5Q/87DDYPspCRshaazTXyGgvd4I+bc05e87V3U6UC2fLKd5MArhIDJM5c6Yh4CEjzmfXNVG3SYjowZdx5q+uZtPkZRJAq6Qv6Ge6GEAAAQQQwP61csMmUpZ9X/rNqKPYEw2j7p+BNARwHek7dM/YDNEk+B9EkwAIIIAAfgY+FZjuIeBJAfAo555cv91Mvs2W2xMWofWbR1btUaeC1ZySM5OHRthRA5i5GgEGAJmoHqaNWADvAAAAAElFTkSuQmCC');
//}
//.controls button[data-state="go-fullscreen"] {
//  background-image: url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACgAAAAoCAYAAACM/rhtAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAA2ZpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuMy1jMDExIDY2LjE0NTY2MSwgMjAxMi8wMi8wNi0xNDo1NjoyNyAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wTU09Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9tbS8iIHhtbG5zOnN0UmVmPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvc1R5cGUvUmVzb3VyY2VSZWYjIiB4bWxuczp4bXA9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC8iIHhtcE1NOk9yaWdpbmFsRG9jdW1lbnRJRD0ieG1wLmRpZDpFNDZDNDg2MEEzMjFFMjExOTBEQkQ4OEMzRUMyQjhERCIgeG1wTU06RG9jdW1lbnRJRD0ieG1wLmRpZDpCM0M2OUNCREE0MDcxMUUyQjgwQkYzQzhCMDZBRTU1NCIgeG1wTU06SW5zdGFuY2VJRD0ieG1wLmlpZDpCM0M2OUNCQ0E0MDcxMUUyQjgwQkYzQzhCMDZBRTU1NCIgeG1wOkNyZWF0b3JUb29sPSJBZG9iZSBQaG90b3Nob3AgQ1M2IChXaW5kb3dzKSI+IDx4bXBNTTpEZXJpdmVkRnJvbSBzdFJlZjppbnN0YW5jZUlEPSJ4bXAuaWlkOjQzQ0QwNDBBMDJBNEUyMTFCOTZEQzYyRDgyRUVBOUZDIiBzdFJlZjpkb2N1bWVudElEPSJ4bXAuZGlkOkU0NkM0ODYwQTMyMUUyMTE5MERCRDg4QzNFQzJCOEREIi8+IDwvcmRmOkRlc2NyaXB0aW9uPiA8L3JkZjpSREY+IDwveDp4bXBtZXRhPiA8P3hwYWNrZXQgZW5kPSJyIj8+cU+iTAAAAZBJREFUeNrs2D9OwzAUBvC6iMMgBgYGpLIxIqYS0cPQKkOIuBErZ2DiDnQoHVBpaQnPEpUs13+e/T5LHfKkb6mr+BfHbh2rrusGx1yqB0pLA83ojyKpBd09xa5/4EkE1oAxaUoBa+CDa7jAIfOCv5R3IPCVskXPQX3BBwDuhvKFesRrMNKF2+UC9Zy7onw6kBMQTg/ArTknuUBzQSCQXpzRb8MFularBBkaObvvhgP0VQ4yiHMtUgkwFRnFlQCGkFUqrhTQhZxTLoz2e8omhisJNJE2zkZ6cRzgwXZLKZWK/Ka8edo1ckV5CQGT9oOJQMh2L1TDwZGXFDimXAbazykj9I6aW9X/Ilh4kBr3QVlSrnMXSS6wsn5GbOQet2/3IksAbZwLMGZ8pwhQ4344HTNvBAr04UaS0c4BtiAcC2n1/cjdsLYgXBRp9DtN3fK3IBxnJKe5L007EC6EXKFe3JfifwU/UgzcgHAmcos8WTil3AGBZ5STEqdbzwDcrOTplhQ5Sz1+609Ye2APjNSfAAMAv4p3Pa/O/tsAAAAASUVORK5CYII=');
//}
//.controls button[data-state="cancel-fullscreen"] {
//  background-image: url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACgAAAAoCAYAAACM/rhtAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAA2ZpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuMy1jMDExIDY2LjE0NTY2MSwgMjAxMi8wMi8wNi0xNDo1NjoyNyAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wTU09Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9tbS8iIHhtbG5zOnN0UmVmPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvc1R5cGUvUmVzb3VyY2VSZWYjIiB4bWxuczp4bXA9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC8iIHhtcE1NOk9yaWdpbmFsRG9jdW1lbnRJRD0ieG1wLmRpZDpFNDZDNDg2MEEzMjFFMjExOTBEQkQ4OEMzRUMyQjhERCIgeG1wTU06RG9jdW1lbnRJRD0ieG1wLmRpZDpCM0M2OUNCOUE0MDcxMUUyQjgwQkYzQzhCMDZBRTU1NCIgeG1wTU06SW5zdGFuY2VJRD0ieG1wLmlpZDpCMzlFNDkzMUE0MDcxMUUyQjgwQkYzQzhCMDZBRTU1NCIgeG1wOkNyZWF0b3JUb29sPSJBZG9iZSBQaG90b3Nob3AgQ1M2IChXaW5kb3dzKSI+IDx4bXBNTTpEZXJpdmVkRnJvbSBzdFJlZjppbnN0YW5jZUlEPSJ4bXAuaWlkOjQzQ0QwNDBBMDJBNEUyMTFCOTZEQzYyRDgyRUVBOUZDIiBzdFJlZjpkb2N1bWVudElEPSJ4bXAuZGlkOkU0NkM0ODYwQTMyMUUyMTE5MERCRDg4QzNFQzJCOEREIi8+IDwvcmRmOkRlc2NyaXB0aW9uPiA8L3JkZjpSREY+IDwveDp4bXBtZXRhPiA8P3hwYWNrZXQgZW5kPSJyIj8+1VELOwAAAadJREFUeNrs2EtOwzAQBuAEOAEIKsQluAAbGqSGHRIrEKveoQEWNOF0SFyGZ6GkwowlWwpm7IwdT8kiI/2LWk39yY4faiqESPpcG0nPq/fARE5xM0btrIGw7fQ4gJeQJ8gRI24MeYac+wIvICv5fcgrE1Li3lUftUZSgAeQhXpQMCGbOJ03yC51BM8gSyYkhpN95b7voA+yQtpukLbMgjsNWSRUZKXa/2wQBjJzjVwosA1ZNdowoMwtFRcKtCHNzzagsDybk/ZlItCGpAJJuK5AjVx1ANYuHAb0PYsPIZsdtpgt9RvRzuIEWa1dp1hYtqBOU3zf0qEvUK/uVmBqotI0/ffb1XBhHYBIlQyL5Dr2NlNGBBZcJ0kZAVhwniQP6qgLrS/II9dJMoF8RhhBee06jj3FGK72ANYIchwLiOFkByeQOQFYqCv9koL0BeYOnK65AzgzfgtDZqFACk7XHdI2Q9pakVTgxAPnW7lruinAfcgHE86FfIHsUUdwCvlmwmFIubde+b6DU/V3BAeuiVxoXMgqHq3hwjLyulEP98EBOAB/148AAwA7RI/R8UopbwAAAABJRU5ErkJggg==');
//}
/*----------- Big Play Button -----------*/
.video-js .vjs-picture-in-picture-control{
  background-image: url('https://i.ibb.co/vdSFsHV/pip.png');
  background-repeat: no-repeat;
  background-size: 22px;
  background-position: 51% calc(50% - 2px);
  border: none !important;
  box-shadow: none !important;
}
.video-js .vjs-big-play-button .vjs-icon-placeholder:before {
  content: "";
  background-image: url('https://i.ibb.co/xsD6VNm/pnghut-music-icon-play-button-gadget-technology.png');
  background-repeat: no-repeat;
  background-size: 46px;
  background-position: 55% calc(50% - 0px);
  border: none !important;
  box-shadow: none !important;
}

.vjs-quality-selector.vjs-menu-button.vjs-menu-button-popup.vjs-control.vjs-button {
  background-image: url('https://i.ibb.co/PYftzKz/setting.png');
  background-repeat: no-repeat;
  background-size: 19px;
  background-position: 55% calc(50% - 2px);
  border: none !important;
  box-shadow: none !important;
}
.vjs-play-control.vjs-control.vjs-button.vjs-paused{
  background-image: url('https://i.ibb.co/z48vSyS/play.png');
  background-repeat: no-repeat;
  background-size: 15px;
  background-position: 55% calc(50% - 0px);
  border: none !important;
  box-shadow: none !important;
}
.vjs-play-control.vjs-control.vjs-button.vjs-playing {
  background-image: url('https://i.ibb.co/ZBXrgRF/pause.png');
  background-repeat: no-repeat;
  background-size: 17px;
  background-position: 55% calc(50% - 0px);
  border: none !important;
  box-shadow: none !important;
}
.video-js .vjs-volume-panel.vjs-volume-panel-vertical {
}
//.video-js .vjs-volume-panel.vjs-volume-panel-vertical .vjs-mute-control {

//}
.video-js .vjs-fullscreen-control {
  background-image: url('https://i.ibb.co/hZ0x7Fg/full.png');
  background-repeat: no-repeat;
  background-size: 17px;
  background-position: 55% calc(50% - 2px);
  border: none !important;
  box-shadow: none !important;
}
.vjs-icon-play:before, .video-js .vjs-play-control .vjs-icon-placeholder:before, .video-js .vjs-big-play-button .vjs-icon-placeholder:before {
  content: "";
}
.vjs-icon-pause:before, .video-js .vjs-play-control.vjs-playing .vjs-icon-placeholder:before {
  content: "";
}
.vjs-icon-volume-mute:before, .video-js .vjs-mute-control.vjs-vol-0 .vjs-icon-placeholder:before {
  content: "";
  background-image: url('https://i.ibb.co/n6rXsFx/mute.png');
  background-repeat: no-repeat;
  background-size: 17px;
  background-position: 55% calc(50% - 0px);
  border: none !important;
  box-shadow: none !important;
}
.vjs-icon-volume-low:before, .video-js .vjs-mute-control.vjs-vol-1 .vjs-icon-placeholder:before {
  content: "";
}
.vjs-icon-volume-mid:before, .video-js .vjs-mute-control.vjs-vol-2 .vjs-icon-placeholder:before {
  content: "";
}
.vjs-icon-volume-high:before, .video-js .vjs-mute-control .vjs-icon-placeholder:before {
  content: "";
  background-image: url('https://i.ibb.co/f8PzV93/volume.png');
  background-repeat: no-repeat;
  background-size: 17px;
  background-position: 55% calc(50% - 0px);
  border: none !important;
  box-shadow: none !important;
}
.vjs-icon-fullscreen-enter:before, .video-js .vjs-fullscreen-control .vjs-icon-placeholder:before {
  content: "";
}
.vjs-icon-fullscreen-exit:before, .video-js.vjs-fullscreen .vjs-fullscreen-control .vjs-icon-placeholder:before {
  content: "";
}
.vjs-icon-picture-in-picture-enter:before, .video-js .vjs-picture-in-picture-control .vjs-icon-placeholder:before {
  content: "";
}
.vjs-icon-picture-in-picture-exit:before, .video-js.vjs-picture-in-picture .vjs-picture-in-picture-control .vjs-icon-placeholder:before {
  content: "";
}
#my-video .vjs-big-play-button {
  /*background-image:url('/src/assets/videoPlayerIcons/1/play.jpg');*/
  border: none;
  width: 100px;
  height: 100px;
  line-height: 2em;
  top: 48%;
  margin-left: -0.5em;
  color: #fff;
  opacity: 0.6;
  background-color: #ff8f00;
  font-size: 52px;
  border-radius:100px;
  -moz-border-radius:100px;
  -webkit-border-radius:100px;
}

#my-video:hover .vjs-big-play-button {
  opacity: 0.7;
}

/*#my-video .vjs-big-play-button:hover {*/
/*  background-color: #ff8f00;*/
/*  color: #fff;*/
/*}*/

/*!*----------------------------------------*!*/

/*!*-------------- Control Bar -------------*!*/
#my-video .vjs-control-bar {
  color: #ccc;
  width: 92%;
  height: 44px;
  left: 1%;
  bottom: 4%;
  font-size: 1.3em;
  //opacity: 0.6; /////////////////******************************its overwriting the opacity of video js
  border-radius: 14px;
  background-color: rgba(0, 0, 0, 0.6);
}
//.vjs-has-started.vjs-user-inactive.vjs-playing .vjs-control-bar {
//  visibility: visible;
//  opacity: 0 !important;
//  transition: visibility 1s, opacity 1s;
//}
//.vjs-default-skin.vjs-has-started.vjs-user-inactive.vjs-playing .vjs-control-bar {
//  display: block;
//  visibility: hidden;
//  opacity: 0;
//  //
//  //@trans: visibility 1.0s, opacity 1.0s;
//  //.transition(@trans);
//}
#my-video .vjs-progress-control .vjs-progress-holder {
  margin: 0 10px;
  height: 24px;
  border-radius: 7px;
}
#my-video .vjs-progress-holder .vjs-play-progress, .video-js .vjs-progress-holder , .video-js .vjs-progress-holder .vjs-load-progress div {
   border-radius: 7px;
}
#my-video .vjs-play-progress::before {
       font-size: 0.9em;
       z-index: 1;
       display: none;
   }
#my-video .vjs-play-progress, #my-video  .vjs-volume-level {
       background: #ffffff;
}

#my-video .vjs-load-progress {
  background-color: #ffffff;
  opacity: 0.2;
    border-radius: 7px;  ////////// مال اون کرو عه لود شدنشه که تو دیزاین نبود ولی من ترجیح میدم بزارم
 }

#my-video .video-js .vjs-progress-holder .vjs-play-progress, .video-js .vjs-progress-holder .vjs-load-progress, .video-js .vjs-progress-holder .vjs-load-progress div {
    //display: none; ////////// یا اینکه کلا هایدش کنم
  background-color: #ffffff;
  opacity: 0.2;
}

 #my-video .vjs-menu-content {
   border-radius: 10px;
   background-color: rgba(0, 0, 0, 0.6);
   padding: 10px 0;
 }

  #my-video .vjs-playback-rate .vjs-menu li {
    text-align: left;
    font-size: 12px;
    padding: 3px 0 3px 11px;
  }
  #my-video .vjs-quality-selector .vjs-menu li {
    text-align: left;
    font-size: 12px;
    padding: 3px 0 3px 15px;
  }
  #my-video .vjs-selected {
    background-color: #fff;
    color: #000000;
  }
   //#my-video .vjs-menu-button-popup .vjs-menu{
   //    bottom: 1em; ///// خیلی خطریه باعث میشه کاربر دیک نتونه برسه ب پلیر
   //    left: 1em;
   //}

#my-video .vjs-playback-rate  .vjs-menu-content {
  padding: 11px 0 !important;
  width: 53px;
}

.vjs-menu-button-popup .vjs-menu .vjs-menu-content {
  bottom: 1.9em;
}
.video-js .vjs-time-control {
  font-size: 12px ;
  font-weight: bold ;
  line-height: 3.75em ;
}
#my-video .vjs-playback-rate .vjs-playback-rate-value {
  line-height: 3.1;
  font-size: 14px;
  font-weight: bold;
}
#my-video .vjs-menu-button-popup .vjs-menu {
  width: 80px;
  left: 0em;
  height: 0em;
  bottom: 1px;
}
#my-video .vjs-quality-selector .vjs-menu {
  width: 80px;
  left: 0em;
  height: 0em;
  bottom: 4px;
}
// .vjs-menu-button-popup .vjs-menu {
//  width: 80px;
//  left: 1em;
//}
/*!* */
// -----------------------buttons-------------------------------------
//#my-video .vjs-icon-volume-high, .video-js .vjs-mute-control .vjs-icon-placeholder {
//  background-image: url('/src/assets/videoPlayerIcons/1/setting.svg');
//}
//#my-video .video-js.video-js-custom .vjs-play-control:before {
//  background-image: url('/src/assets/videoPlayerIcons/1/setting.svg');
//}
//#my-video .video-js .vjs-quality-selector .vjs-menu-button{
//  position: relative;
//  right: -8px;
//
//}

/* *!*/
/*!*--------------------------------------------------------------------------------------*!*/
//#my-video .vjs-progress-control:hover .vjs-time-tooltip { display: none;}

/*#my-video .vjs-progress-control:hover .vjs-play-progress .vjs-time-tooltip { display: block;} *!*/

/*!* #my-video .vjs-progress-control:hover .vjs-time-tooltip, .video-js .vjs-progress-control:hover .vjs-progress-holder:focus .vjs-time-tooltip{*/
/*!* font-size: 0; *!*/


/*!*----------- Play/Pause Toggle ----------*!*/

   #my-video .vjs-play-control {

   }

/*!*----------- Fullscreen Toggle ----------*!*/

/*!* #my-video .vjs-fullscreen-control {*/
/*  left: 102%;*/
/*  background: #262626;*/
/*  position: absolute;*/
/*  border-radius: 10%;*/
/*}*/
/* *!*/
/* !*---------- Add/Remove Button ----------*!*/

/*!* #my-video .vjs-picture-in-picture-control {*/
/*  display: none;*/
/*} *!*/


/*!*----------- Volume Vertical ------------*!*/

   #my-video .vjs-volume-vertical {
       //background: #262626;
       font-size: 1.1em;
       border-radius: 10px;
       background-color: rgba(0, 0, 0, 0.6);
       position: relative;
       left: -28px;
       bottom: 103px;
       width: 32px;
       height: 100px;
   }

   #my-video .vjs-slider {
       border-radius: 10px;
       background-color: rgba(255, 255, 255, 0.2);
   }
   #my-video .vjs-volume-bar.vjs-slider-vertical {
       width: 32px;
       height: 32px;
       margin: 1.35em auto;
       background-color: rgba(0, 0, 0, 0.1);
   }
   #my-video .vjs-slider-vertical .vjs-volume-level {
       //width: 0.3em;
       width: 32px;
       border-radius:10px;
   }
   #my-video .video-js .vjs-volume-panel .vjs-volume-control.vjs-volume-vertical {
       width: 32px;
       height: 100px;
   }
   #my-video .vjs-volume-bar.vjs-slider-vertical {
       width: 32px;
       height: 100px;
       margin-top: 0;
   }
   //#my-video .vjs-volume-vertical {
   //    //background: #262626;
   //    font-size: 1.1em;
   //    border-radius: 10px;
   //    background-color: rgba(0, 0, 0, 0.6);
   //    position: relative;
   //
   //}
   #my-video .vjs-volume-vertical:hover .vjs-volume-tooltip {
     left : 35px;
   }
   #my-video .vjs-volume-level::before {
     display: none;
   }
   #my-video .vjs-current-time{
     position: relative;
     width: 40px;
     padding: 0;
     z-index: 1;
     color: #fff;
     display: block;
     right: -4px;
   }

   #my-video .vjs-remaining-time {
     position: relative;
   color: #fff;
     width: 46px;
     padding: 0;
     padding-right: 10px;
   }

   #my-video .vjs-button:hover {
     color: #ffffff;
     opacity: 1;
   }

   #my-video .vjs-control:focus:before,
   #my-video .vjs-control:hover:before,
   #my-video .vjs-control:focus {
     text-shadow: none;
   }
.video-js .vjs-control {
  width: 25px;
}
.video-js .vjs-volume-panel.vjs-volume-panel-vertical {
  width: 30px;
  padding-top: 2px;
}
.video-js .vjs-fullscreen-control {
  padding-left: 30px;
  padding-top: 2px;
}
.vjs-quality-selector.vjs-menu-button.vjs-menu-button-popup.vjs-control.vjs-button {
  padding-right: 40px;
  padding-top: 2px;
  position: relative;
  top: 2.75px;
}
.video-js .vjs-picture-in-picture-control {
  cursor: pointer;
  flex: none;
  width: 30px !important;
  padding-right: 48px;
  position: relative;
  top: 2.75px;

}
.video-js .vjs-fullscreen-control {
  width: 25px !important;
  position: relative;
  top: 2.75px;
}
.vjs-playback-rate.vjs-menu-button.vjs-menu-button-popup.vjs-control.vjs-button {
  width: 30px;
}
.vjs-play-control.vjs-control.vjs-button.vjs-paused {
  width: 35px;
  position: relative;
  top: 2px;
  left: 3px;
}
.vjs-play-control.vjs-control.vjs-button.vjs-playing {
  width: 35px;
  position: relative;
  top: 2px;
  left: 3px;
}
//.video-js .vjs-control .vjs-remaining-time{
//  width: 60px;
//}
@media screen and (max-width: 708px) {
  #my-video .vjs-playback-rate .vjs-playback-rate-value {
    line-height: 2.75;
    font-size: 14px;
  }
  .vjs-quality-selector.vjs-menu-button.vjs-menu-button-popup.vjs-control.vjs-button {
    padding-right: 10px;
  }
  .video-js .vjs-picture-in-picture-control {
    width: 10px !important;
    position: relative;
    right: 4px;
    padding-right: 44px;
  }
  .video-js .vjs-fullscreen-control {
    right: -2px;
  }
  .vjs-playback-rate.vjs-menu-button.vjs-menu-button-popup.vjs-control.vjs-button {
    position: relative;
    top: 2.75px;
    right: -6px;
  }
  .vjs-play-control.vjs-control.vjs-button.vjs-paused {
    left: 7px;
  }
  #my-video .vjs-current-time{
    right: -4px;
  }
}
@media screen and (max-width: 960px) {}
@media screen and (max-width: 350px) {}
//@media screen and (max-width: 1200px) {}
/*!* position: relative;*/
/*left: 63px;*/
/*z-index: 1;*/
/*color: #000; *!*/
</style>
// ----------------------------------------------------------------------------------------------------------------
<style lang="scss">
.vPlayer-timestamp-icon{
    background: url('https://i.ibb.co/Ss3KVVw/Asset-25.png');
    height: 21px;
    width: 23px;
    display: block;
    /* Other styles here */
}
.vPlayer-drawer-btn {
  position: absolute !important;
  right: 0 !important;
  z-index: 4 !important;
  bottom: 23px;
  margin-right: 13px;
}
.vPlayer .v-btn:not(.v-btn--round).v-size--default {
  height: 44px;
  min-width: 44px;
  padding: 0 ;
  border-radius: 15px;
}
.video-js .vjs-current-time { display: block; }
.icon-btn-box .video-box-icon .favorite-bookmark {
    color: #ff8f00;
}
.vPlayer .is-favorite{
    color: #ff8f00;
}
.vPlayer .is-not-favorite{
    color: #3e5480;
}
.vPlayer .bookmark-button {
    padding-left: 2px;
    padding-top: 12px;
    font-size: 20px;
}
.vPlayer-drawer-btn{
    position: absolute !important;
    right: 0 !important;
    z-index: 4 !important;
}
.vPlayer .theme--light.v-navigation-drawer {
    background-color: rgba(0, 0, 0, 0.6);
    z-index: 4;
}
.vPlayer v-list-item .v-list-item__subtitle, .v-list-item .v-list-item__title {
    font-size: 18px;
    font-weight: 500;
    color: #3e5480;
    padding-right: 8px;
}
.vPlayer .v-list--dense .v-list-item .v-list-item__subtitle, .v-list--dense .v-list-item .v-list-item__title, .v-list-item--dense .v-list-item__subtitle, .v-list-item--dense .v-list-item__title {
    font-weight: 500;
    font-size: 18px;
    padding-top: 8px;
    padding-bottom: 8px;
    padding-right: 0;
}
.vPlayer .v-list--nav .v-list-item, .v-list--nav .v-list-item::before {
    padding-bottom: 0;
    padding-right: 0px;
}
.vPlayer .v-list--nav.v-list--dense .v-list-item:not(:last-child):not(:only-child), .v-list--nav .v-list-item--dense:not(:last-child):not(:only-child), .v-list--rounded.v-list--dense .v-list-item:not(:last-child):not(:only-child), .v-list--rounded .v-list-item--dense:not(:last-child):not(:only-child) {
    margin-bottom: 4px;
    padding-bottom: 0;
    padding-right: 0;
}
@media screen and (max-width: 1200px) {
    .vPlayer .v-navigation-drawer--is-mobile:not(.v-navigation-drawer--close), .v-navigation-drawer--temporary:not(.v-navigation-drawer--close) {
        width: 220px !important;
      height: 518px;
    }
    .vPlayer .v-btn--fab.v-size--default {
        height: 45px;
        width: 45px;
    }
    .vPlayer v-list-item .v-list-item__subtitle, .v-list-item .v-list-item__title {
        font-size: 16px;
    }
    .vPlayer .v-list--dense .v-list-item .v-list-item__subtitle, .v-list--dense .v-list-item .v-list-item__title, .v-list-item--dense .v-list-item__subtitle, .v-list-item--dense .v-list-item__title {
        font-size: 16px;
    }
    .vPlayer .v-list--nav .v-list-item:not(:last-child):not(:only-child), .v-list--rounded .v-list-item:not(:last-child):not(:only-child) {
        margin-bottom: 0;
        padding-bottom: 0;
    }
    .vPlayer .v-list--dense .v-list-item .v-list-item__subtitle, .v-list--dense .v-list-item .v-list-item__title, .v-list-item--dense .v-list-item__subtitle, .v-list-item--dense .v-list-item__title {
        padding-top: 4px;
        padding-bottom: 4px;
    }
    .vPlayer .v-list--dense .v-list-item, .v-list-item--dense {
        min-height: 35px;
    }
    .video-box-icon-button{
        justify-content: flex-end;
    }
    .vPlayer .bookmark-button {
        padding-left: 9px;
        font-size: 18px;
    }
}
@media only screen and (max-width: 960px){
    //.vPlayer .v-navigation-drawer--is-mobile:not(.v-navigation-drawer--close), .v-navigation-drawer--temporary:not(.v-navigation-drawer--close) {
    //    width: 256px !important;
    //}
  .vPlayer .v-navigation-drawer--is-mobile:not(.v-navigation-drawer--close), .v-navigation-drawer--temporary:not(.v-navigation-drawer--close) {
    width: 180px !important;
    background-color: rgba(0, 0, 0, 0.6);
    position: absolute;
    height: 220px;
    bottom: 4px;
    right: 7px;
    top: 125px !important;
  }
  .v-navigation-drawer--bottom.v-navigation-drawer--is-mobile {
    min-width: 180px !important;
  }
}
@media screen and (max-width: 768px) {
    .vPlayer .v-list--nav .v-list-item:not(:last-child):not(:only-child), .v-list--rounded .v-list-item:not(:last-child):not(:only-child) {
        margin-bottom: 0;
        padding-bottom: 0;
    }
    .vPlayer .v-list--dense .v-list-item .v-list-item__subtitle, .v-list--dense .v-list-item .v-list-item__title, .v-list-item--dense .v-list-item__subtitle, .v-list-item--dense .v-list-item__title {
        padding-top: 4px;
        padding-bottom: 4px;
    }
}
@media screen and (max-width: 576px) {
    .vPlayer .v-navigation-drawer--is-mobile:not(.v-navigation-drawer--close), .v-navigation-drawer--temporary:not(.v-navigation-drawer--close) {
        width: 150px !important;
    }
    .vPlayer .v-btn--fab.v-size--default {
        height: 35px;
        width: 35px;
    }
    .vPlayer v-list-item .v-list-item__subtitle, .v-list-item .v-list-item__title {
        font-size: 12px;
    }
    .vPlayer .v-list--dense .v-list-item .v-list-item__subtitle, .v-list--dense .v-list-item .v-list-item__title, .v-list-item--dense .v-list-item__subtitle, .v-list-item--dense .v-list-item__title {
        font-size: 12px;
    }
    .vPlayer .v-list--dense .v-list-item .v-list-item__subtitle, .v-list--dense .v-list-item .v-list-item__title, .v-list-item--dense .v-list-item__subtitle, .v-list-item--dense .v-list-item__title {
        padding-top: 0;
        padding-bottom: 0;
    }
    .vPlayer .v-list--dense .v-list-item, .v-list-item--dense {
        min-height: 30px;
    }
    .vPlayer .bookmark-button {
        padding-left: 10px;
        padding-bottom: 4px;
        font-size: 15px;
    }

}


</style>
<style scoped lang="scss">
.v-application--is-rtl .v-list-item__action:first-child, .v-application--is-rtl .v-list-item__icon:first-child {
    margin-left: 8px;
    margin-top: 3px;
    margin-bottom: 3px;
}
</style>
