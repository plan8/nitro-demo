<template>
  <div></div>
</template>

<script>
import { mapState } from "vuex";
// import { Player, JCReverb } from "tone";

export default {
  components: {},
  data() {
    return {
      reverbValue: 0,
      seekValue: 0,
      isPlaying: false,
    };
  },
  computed: {
    ...mapState({
      item: (state) => state.player.item,
      useTone: (state) => state.player.useTone,
      currentStem: (state) => state.player.currentStem,
      targetPosition: (state) => state.player.targetPosition,
      playbackStates: (state) => state.player.playbackStates,
    }),
    currentSource() {
      return `${this.$config.previewURL}${this.currentStem.key}.mp3`;
    },
  },
  props: {
    settings: {
      type: Object,
      required: false,
    },
  },
  mounted() {
    this.player = null;

    this.simplePlayer = new Audio();
    this.simplePlayer.preload = true;
    //this.simplePlayer.autoplay = true;2

    this.addListeners();
  },
  watch: {
    // currentSource(e) {
    //

    //   this.simplePlayer.src = e
    // },
    item(e) {
      if (this.useTone) {
        // this.$store.commit("player/SET_PLAYBACKSTATE", "loading");
        this.player.load(this.currentSource).then((r) => {
          this.$store.commit(
            "player/SET_PLAYBACKSTATE",
            this.playbackStates.PAUSED
          );
        });
      } else {
        // this.simplePlayer.pause();
        this.isPlaying = false;
        this.stop();
        this.simplePlayer.src = this.currentSource;

        this.simplePlayer.onloadedmetadata = () => {
          if (this.targetPosition) {
            let pos = this.simplePlayer.duration * this.targetPosition;
            this.simplePlayer.currentTime = pos;
            console.log("har targetposition", this.targetPosition);
            this.toggle();

          }
            
          };

        if (this.targetPosition) {
          
        } else {
          this.toggle();
        }

        // this.$analyticsHandler.track("Play Item", {
        //   itemId: this.item.id,
        //   itemName: this.item.originalName,
        // });
      }
    },
    useTone(val) {
      if (val) {
        const reverb = new JCReverb({ wet: this.reverbValue }).toDestination();

        this.reverb = reverb;

        this.player = new Player().connect(reverb);

        this.player.autostart = true;
      } else {
        this.player = null;
      }
    },
  },
  beforeDestroy() {
    this.$nuxt.$off("stopAll", this.stop);
    this.$nuxt.$off("audio-playing");
    clearInterval(this.updateTimer);
  },
  methods: {
    addListeners() {
      this.$nuxt.$on("stopAll", this.stop);
      // this.simplePlayer.addEventListener("timeupdate", () => {
      //   this.$nuxt.$emit("audio-playing", this.simplePlayer);
      // });
      this.simplePlayer.addEventListener("loadstart", () => {
        // this.$store.commit("player/SET_PLAYBACKSTATE", this.playbackStates.LOADING);
      });

      this.simplePlayer.addEventListener("play", () => {
        this.isPlaying = true;
        this.$store.commit(
          "player/SET_PLAYBACKSTATE",
          this.playbackStates.PLAYING
        );
      });

      this.simplePlayer.addEventListener("canplay", () => {});

      this.simplePlayer.addEventListener("progress", () => {
        this.$store.commit(
          "player/SET_CURRENTTIME",
          this.simplePlayer.currentTime
        );
      });

      this.simplePlayer.addEventListener("pause", () => {
        this.isPlaying = false;
        this.$store.commit(
          "player/SET_PLAYBACKSTATE",
          this.playbackStates.PAUSED
        );
      });

      this.simplePlayer.addEventListener("ended", () => {
        this.$store.commit(
          "player/SET_PLAYBACKSTATE",
          this.playbackStates.PAUSED
        );
      });

      this.$nuxt.$on("audio-seek", ({ position, item }) => {
        if (this.simplePlayer) {
          console.log("this.item: ", this.item);

          if (this.item.stems.length) {
            if (item.stems[0].key !== this.item.stems[0].key) {
              console.log("not the savme");

              this.$store.commit("player/SET_TARGETPOSITION", position);
              console.log("position: ", position);
              this.$store.commit("player/SET_ITEM", item);
            } else {
              let pos = this.simplePlayer.duration * position;
              this.simplePlayer.currentTime = pos;

              if (!this.isPlaying) {
                this.play()
              }
            }
          } else {
            this.$store.commit("player/SET_ITEM", item);
          }

          this.$store.commit(
            "player/SET_PLAYBACKSTATE",
            this.playbackStates.PLAYING
          );
        }
        if (this.useTone) {
          let newTime = this.player.buffer.duration * relative;
          this.player.seek(newTime, "+0.1");
        }
      });

      this.$nuxt.$on("player-pause", () => {
        this.stop();
      });

      this.$nuxt.$on("player-play", () => {
        this.play();
      });

      this.$nuxt.$on("player-toggle", () => {
        this.toggle();
      });

      this.$nuxt.$on("audio-rev", (relative) => {
        if (this.simplePlayer) {
          let currentPos = this.simplePlayer.currentTime;
          let step = this.simplePlayer.duration / 10;
          this.simplePlayer.currentTime = currentPos - step;
        }
      });

      this.$nuxt.$on("audio-ff", (relative) => {
        if (this.simplePlayer) {
          let currentPos = this.simplePlayer.currentTime;
          let step = this.simplePlayer.duration / 10;
          this.simplePlayer.currentTime = currentPos + step;
        }
      });

      this.updateTimer = setInterval(() => {
        if (this.simplePlayer) {
          let progress = Math.max(
            0,
            (this.simplePlayer.currentTime / this.simplePlayer.duration) * 100
          );

          if (this.isPlaying) {
            this.$nuxt.$emit("audio-playing", {
              progress: progress,
              currentTime: this.simplePlayer.currentTime,
            });
          }
        } else if (this.useTone) {
        }
      }, 100);
    },
    changeEffect() {
      //this.reverb.wet.value = this.reverbValue
      this.player.playbackRate = this.reverbValue;
      //
    },
    play() {
      if (!this.item.id) return;
      if (this.useTone) {
        if (this.player.loaded) {
          const time = this.player.now() + 0.1;
          this.player.start(time);
        }
      } else {
        this.simplePlayer.play();
      }
    },
    toggle() {
      if (this.isPlaying) {
        this.stop();
      } else {
        this.play();
      }
    },
    stop() {
      if (this.player) this.player.stop();
      this.simplePlayer.pause();
    },
  },
};
</script>

<style lang="scss" scoped>
</style>
