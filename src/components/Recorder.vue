<template>
  <div>
    <h2>Video Recorder</h2>
    <main>
      <div class="videocontrols">
        <button
          v-if="!permission"
          @click="getCameraPermission"
          type="button"
          class="opencamerabutton"
        >
          Open camera
        </button>
      </div>

      <div class="videoplayer">
        <video v-if="!recordedVideo" ref="liveVideoFeed" autoPlay muted></video>

        <div v-if="permission && recordingStatus === 'inactive'">
          <!--{seconds !== 0 ?
          <p className="{styles.timer}">{seconds}</p>
          : null}-->
          <button @click="startRecording()" type="button" class="recordbutton">
            <img
              style="position: absolute; height: 2.5em"
              src="@/assets/playicon.png"
              alt="startRecording"
            />
          </button>
        </div>

        <div v-if="recordingStatus === 'preparing'">
          <!-- {seconds !== 0 ? <p className={styles.timer}>{seconds}</p> : null}

              <Countdown
                onComplete={startRecording}
                className={styles.countdown}
                date={Date.now() + 3000}
              ></Countdown>-->
        </div>
        <div v-if="recordingStatus === 'recording'">
          <!--<p className="{styles.timer}">{seconds}</p>-->
          <button @click="pauseRecording" type="button" class="pausebutton">
            <img
              style="position: absolute; height: 2.5em"
              src="@/assets/pauseicon.png"
              alt="pauseRecording"
            />
          </button>
          <button @click="stopRecording" type="button" class="recordbutton">
            <img
              style="position: absolute; height: 2.5em"
              src="@/assets/stopicon.png"
              alt="stopRecording"
            />
          </button>
        </div>
        <div v-if="recordedVideo" class="recordedplayer">
          <video className="recorded" :src="recordedVideo" controls></video>
          <a class="opencamerabutton" download :href="recordedVideo">
            Download Recording
          </a>
        </div>
      </div>
    </main>
  </div>
</template>

<script>
import { ref } from "vue";

export default {
  name: "Recorder",
  components: {},
  data() {
    return {
      permission: false,
      mediaRecorder: ref(null),
      recordingStatus: "inactive",
      stream: null,
      videoChunks: [],
      recordedVideo: null,

      //seconds, start, pause, reset, running, stop : useTimer()
    };
  },

  methods: {
    changeRecordingStatus(value) {
      this.recordingStatus = value;
    },
    changeRecordedVideo(value) {
      this.recordedVideo = value;
    },
    changePermission(value) {
      this.permission = value;
    },
    changeStream(value) {
      this.stream = value;
    },
    changeVideoChunks(value) {
      this.videoChunks = value;
    },
    changeMediaRecorder(value) {
      this.mediaRecorder = value;
    },
    async getCameraPermission() {
      this.changeRecordedVideo(null);
      if ("MediaRecorder" in window) {
        try {
          const constraints = {
            audio: true,
            video: true,
          };
          const stream = await navigator.mediaDevices.getUserMedia(constraints);
          const videoConstraints = {
            audio: false,
            video: true,
          };

          const audioConstraints = {
            audio: true,
          };
          // create audio and video streams separately
          const videoStream = await navigator.mediaDevices.getUserMedia(
            videoConstraints
          );
          const audioStream = await navigator.mediaDevices.getUserMedia(
            audioConstraints
          );

          this.changePermission(true);
          //combine both audio and video streams
          const combinedStream = new MediaStream([
            ...audioStream.getTracks(),
            ...videoStream.getTracks(),
          ]);

          this.changeStream(combinedStream);
          //set videostream to live feed player
          this.$refs.liveVideoFeed.srcObject = combinedStream;
        } catch (err) {
          alert(err.message);
        }
      } else {
        alert("The MediaRecorder API is not supported in your browser.");
      }
    },

    async startRecording() {
      var mimeType = "video/webm";
      this.changeRecordingStatus("recording");
      if (!this.mediaRecorder) {
        const media = new MediaRecorder(this.stream, { mimeType });
        this.changeMediaRecorder(media);
        console.log(this.mediaRecorder);
      }
      if (this.mediaRecorder.state === "paused") {
        this.mediaRecorder.resume();
      } else {
        this.mediaRecorder.start();
      }
      //start();
      let localVideoChunks = [];
      this.mediaRecorder.ondataavailable = (event) => {
        if (typeof event.data === "undefined") return;
        if (event.data.size === 0) return;
        localVideoChunks.push(event.data);
      };
      this.changeVideoChunks(localVideoChunks);
    },

    async pauseRecording() {
      this.changeRecordingStatus("inactive");
      this.mediaRecorder.value.pause();
      //pause();
    },

    stopRecording() {
      this.changePermission(false);
      this.changeRecordingStatus("inactive");

      if (this.mediaRecorder) {
        // Check if mediaRecorder.value is not null
        this.mediaRecorder.stop();

        this.mediaRecorder.onstop = () => {
          const videoBlob = new Blob(this.videoChunks, { type: "video/webm" }); // Use the correct mimeType
          const videoUrl = URL.createObjectURL(videoBlob);
          this.changeRecordedVideo(videoUrl);
          this.changeVideoChunks([]);
        };
      }
    },
  },
};
</script>

<style>
main {
  position: relative;
}
.opencamerabutton {
  border: none;
  height: 25px;
  background: linear-gradient(#fe623d, #ff3472);
  font-size: 1.2em;
  color: white;
  margin-bottom: 1em;
  border-radius: 10px;
  padding: 0.4em;
  padding-bottom: 1.2em;
}
.recordbutton {
  border: none;
  padding: 0;
  background-color: transparent;
  position: absolute;
  bottom: -17.5em;
}
.pausebutton {
  border: none;
  padding: 0;
  background-color: transparent;
  position: absolute;
  bottom: -17.5em;
  left: 11.5em;
}
.logo {
  width: 50p;
  height: 50px;
  float: left;
}

.videocontrols {
  margin-bottom: 20px;
}
.videoplayer {
  display: flex;
  flex-direction: column;
  align-items: center;
  position: relative;
}
.videoplayer,
.recordedplayer {
  display: flex;
  flex-direction: column;
  align-items: center;
  height: 200px;
  width: 400px;
  margin: auto;
}
.liveplayer {
  height: 400px;
  width: 800px;
  margin-bottom: 30px;
}
.recordedplayer video {
  height: 400px;
  width: 800px;
  margin-bottom: 1em;
}
</style>
