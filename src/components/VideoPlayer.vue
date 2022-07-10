<template>
  <div v-if="isSupported">
    <div>
      <label for="src">Source:</label>
      <input @change="changeSource" type="text" id="src" v-model="src" />
    </div>
    <video ref="video" width="100%" class="video" controls></video>
    <div class="grid">
      <div class="grid-item">
        <h4>Total time played</h4>
        <pre>{{ totalPlayedTime }} s</pre>
        <h4>Total buffer length</h4>
        <pre>{{ totalBufferedLength }} s</pre>
      </div>
      <div class="grid-item">
        <h4>Status</h4>
        <pre ref="status" class="status">{{ logs }}</pre>
      </div>
    </div>
  </div>
  <div v-else class="err-msg">
    Sorry, your browser doesn't support MediaSource Extensions.
  </div>
</template>

<script>
import Vue from "vue";
import Hls from "hls.js";

export default {
  name: "VideoPlayer",
  props: {
    msg: String,
  },
  data() {
    return {
      logs: "",
      src: "https://live-streams.cdnvideo.ru/cdnvideo/caminandes/playlist.m3u8",
      hls: null,
      played: null,
      buffered: null,
    };
  },
  mounted() {
    if (this.isSupported) {
      this.addSource(this.src);
      let video = this.$refs.video;
      video.addEventListener("timeupdate", (event) => {
        this.updateProp(event.target, "played");
      });
      video.addEventListener("progress", (event) => {
        this.updateProp(event.target, "buffered");
      });
    }
  },
  computed: {
    isSupported() {
      return Hls.isSupported();
    },
    totalPlayedTime() {
      return this.sumTimeRanges("played");
    },
    totalBufferedLength() {
      return this.sumTimeRanges("buffered");
    },
  },
  methods: {
    addLog(log) {
      this.logs += `${log}\n`;
      this.scrollToLastLog();
    },
    scrollToLastLog() {
      Vue.nextTick(
        () => (this.$refs.status.scrollTop = this.$refs?.status.scrollHeight)
      );
    },
    changeSource() {
      this.hls.destroy();
      this.addLog("Media element detached");
      this.addSource(this.src);
    },
    addSource(src) {
      this.hls = new Hls();
      this.hls.attachMedia(this.$refs.video);
      this.hls.on(Hls.Events.MEDIA_ATTACHED, () => {
        this.addLog("video and hls.js are now bound together");
        this.addLog(`Loading ${this.src}`);
        this.hls.loadSource(src);
        this.hls.on(Hls.Events.MANIFEST_PARSED, (event, data) => {
          this.addLog(
            "manifest loaded, found " + data.levels.length + " quality level"
          );
        });
      });
      this.handleError();
    },
    handleError() {
      this.hls.on(Hls.Events.ERROR, (event, data) => {
        if (data.fatal) {
          switch (data.type) {
            case Hls.ErrorTypes.NETWORK_ERROR:
              this.addLog("fatal network error encountered, try to recover");
              this.hls.startLoad();
              break;
            case Hls.ErrorTypes.MEDIA_ERROR:
              this.addLog("fatal media error encountered, try to recover");
              this.hls.recoverMediaError();
              break;
            default:
              this.hls.destroy();
              break;
          }
        }
      });
    },
    updateProp(target, prop) {
      this[prop] = target[prop];
    },
    sumTimeRanges(prop) {
      let total = 0;
      if (this[prop]) {
        for (let i = 0; i < this[prop].length; i++) {
          total += this[prop].end(i) - this[prop].start(i);
        }
      }
      return total.toFixed(3);
    },
  },
};
</script>

<style scoped lang="scss">
label {
  display: block;
}
input {
  display: block;
  padding: 4px 8px;
  line-height: 1.6;
  margin: 10px 0;
  border: 1px solid #ccc;
  border-radius: 4px;
  width: 100%;
}
.grid {
  display: grid;
  grid: auto / 1fr 1fr;
  grid-gap: 40px;
}
.status {
  height: 100px;
  overflow: auto;
}
.err-msg {
  font-size: 30px;
  font-weight: 900;
}
@media (min-width: 1200px) {
  .video {
    min-height: 600px;
  }
}
@media (max-width: 1199px) {
  .grid {
    display: block;
  }
}
</style>
