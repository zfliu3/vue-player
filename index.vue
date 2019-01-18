<template>
  <Row class="vue-player">
    <Col span="24">
    <Card :border="false">
      <Slider
        :disabled="!loaded"
        v-model="percentage"
        :tip-format="processTipFormat"
        @on-input="setPosition()"></Slider>

      <Button
        :disabled="!loaded"
        :icon="`${ !playing||paused?'play':'pause' }`"
        :title="`${ !playing||paused?'播放':'暂停' }`"
        class="btn-control"
        size="large"
        type="ghost"
        shape="circle"
        @click="()=>{playing ? pause() : play()}"
      ></Button>
      <Button
        :disabled="!loaded"
        class="btn-control"
        icon="stop"
        size="large"
        title="停止"
        type="ghost"
        shape="circle"
        @click="stop()">
      </Button>
      <Button
        :disabled="!loaded"
        :icon="`${!isMuted ? volumeIcon:'volume-mute'}`"
        class="btn-control"
        size="large"
        title="音量"
        type="ghost"
        shape="circle"
        @click="mute()">
      </Button>
      <div class="volume">
        <Slider
          :disabled="!loaded"
          v-model="volumeValue"
          :tip-format="()=>{return null}"
          @on-input="setVolumeValue()"></Slider>
      </div>
      <span style="margin-left: 20px;">{{ currentTime }} / {{ duration }}</span>
      <audio
        ref="player"
        :src="audioUrl"
        @ended="ended"
        @canplay="canPlay"></audio>
    </Card>
    </Col>
  </Row>
</template>
<script>
const formatTime = second => new Date(second * 1000).toISOString().substr(11, 8);
export default {
  name: 'VuePlayer',
  props: {
    url: {
      type: String,
      default: null
    },
    autoPlay: {
      type: Boolean,
      default: false
    },
    ended: {
      type: Function,
      default: () => { },
    },
    canPlay: {
      type: Function,
      default: () => { },
    },
  },
  data() {

    return {
      audioUrl: '',
      firstPlay: true,
      isMuted: false,
      loaded: false,
      playing: false,
      paused: false,
      percentage: 0,
      currentTime: '00:00:00',
      audio: undefined,
      totalDuration: 0,
      volumeValue: 75
    }

  },
  computed: {
    volumeIcon: function () {

      if (this.volumeValue === 0) return 'volume-mute';
      return this.volumeValue < 35 ? 'volume-low' : (this.volumeValue > 70 ? 'volume-high' : 'volume-medium');
    },
    duration: function () {

      return this.audio ? formatTime(this.totalDuration) : ''

    },
  },
  watch: {
    'url': function (val) {

      this.audioUrl = val;
      if (this.audio) {

        this.audio.currentTime = 0;
        this.percentage = 0;
        this.playing = false;

      }

      //音频地址为空时，按钮控件全置灰
      if (!this.audioUrl) {

        this.loaded = false;
        this.totalDuration = 0;
      }
    }
  },
  mounted() {

    this.audioUrl = this.url;
    this.audio = this.$refs.player;
    this.init();

  },
  beforeDestroy() {

    this.audio.removeEventListener('timeupdate', this.handlePlayingUI)
    this.audio.removeEventListener('loadeddata', this.handleLoaded)
    this.audio.removeEventListener('pause', this.handlePlayPause)
    this.audio.removeEventListener('play', this.handlePlayPause)
    this.audio.removeEventListener('ended', this.handleEnded);

  },
  methods: {
    setVolumeValue() {

      if (!this.volumeValue) this.isMuted = true;
      this.audio.volume = this.volumeValue / 100.0;
    },
    setPosition() {

      if (this.audio && this.audio.duration) {

        //进度值变化引起的时间变化小于 this.audio.duration / 100 s时，忽略移动
        if (Math.abs(this.audio.currentTime - this.percentage * this.audio.duration / 100.0) < this.audio.duration / 100) return;
        this.audio.currentTime = this.audio.duration / 100.0 * this.percentage;
      }
    },
    stop() {

      this.paused = this.playing = false
      this.audio.pause()
      this.audio.currentTime = 0

    },
    play() {

      if (this.playing) return
      this.paused = false
      this.audio.play();

      let _this = this;
      this.$nextTick(() => {

        _this.playing = true

      })

    },
    pause() {

      this.paused = !this.paused;
      (this.paused) ? this.audio.pause() : this.audio.play()

    },
    mute() {

      this.isMuted = !this.isMuted;
      this.audio.muted = this.isMuted;
      this.volumeValue = this.isMuted ? 0 : 75;
      this.audio.volume = this.volumeValue / 100.0;

    },
    reload() {

      this.audio.load();

    },
    handleLoaded: function () {

      if (this.audio.readyState >= 2) {

        if (this.audio.duration === Infinity) {

          // Fix duration for streamed audio source or blob based
          // https://stackoverflow.com/questions/38443084/how-can-i-add-predefined-length-to-audio-recorded-from-mediarecorder-in-chrome/39971175#39971175
          this.audio.currentTime = 1e101;
          this.audio.ontimeupdate = () => {

            this.audio.ontimeupdate = () => { }
            this.audio.currentTime = 0
            this.totalDuration = parseInt(this.audio.duration)
            this.loaded = true;

          }

        } else {

          this.totalDuration = parseInt(this.audio.duration);
          this.loaded = true;
        }

        //自动播放音频
        if (this.loaded && this.autoPlay) {

          let _this = this;
          this.$nextTick(() => {

            _this.playing = true;
            _this.audio.play();
          });
        }

      } else {

        throw new Error('Failed to load sound file')

      }

    },
    handlePlayingUI: function () {

      this.percentage = this.audio.currentTime * 100.0 / this.audio.duration;
      this.currentTime = formatTime(this.audio.currentTime);

    },
    handlePlayPause: function (e) {

      if (e.type === 'play' && this.firstPlay) {

        // in some situations, audio.currentTime is the end one on chrome
        this.audio.currentTime = 0;
        if (this.firstPlay) {

          this.firstPlay = false;

        }

      }
      if (e.type === 'pause' && this.paused === false && this.playing === false) {

        this.currentTime = '00:00:00'

      }

    },
    handleEnded() {

      this.paused = this.playing = false;

    },
    init: function () {

      this.audio.addEventListener('timeupdate', this.handlePlayingUI);
      this.audio.addEventListener('loadeddata', this.handleLoaded);
      this.audio.addEventListener('pause', this.handlePlayPause);
      this.audio.addEventListener('play', this.handlePlayPause);
      this.audio.addEventListener('ended', this.handleEnded);

    },
    processTipFormat: function () {

      return this.currentTime;
    }
  }
}
</script>
<style  lang="scss">
.vue-player {
  text-align: center;
  padding-top: 10px;
  .btn-control {
    color: #0094ff;
    margin-left: 15px;
    &:disabled {
      color: #bbbec4;
    }
  }
  .volume {
    width: 63px;
    display: inline-block;
    margin-left: 10px;
    vertical-align: middle;
  }
}
</style>
