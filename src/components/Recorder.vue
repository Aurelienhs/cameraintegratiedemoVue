<template>
  <div>
    <h2>Video Recorder</h2>
    <main>
      <div>
        <button v-if="!permission" @click="getCameraPermission" type="button">
          Open camera
        </button>
      </div>

      <div>
        <video v-if="!recordedVideo" ref="liveVideoFeed" autoPlay muted></video>

        <div v-if="permission && recordingStatus === 'inactive'">
          <!--{seconds !== 0 ?
          <p className="{styles.timer}">{seconds}</p>
          : null}-->
          <button
            @click="startRecording()"
            type="button"
            className="{styles.recordbutton}"
          >
            <p>Start recording</p>
            <!--<img
              height="30em"
              position="absolute"
              src="{playIcon}"
              alt="startRecording"
            />-->
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
          <button @click="pauseRecording" type="button">
            <p>pause</p>
            <!-- <img height="30em" src="{pauseIcon}" alt="pauseRecording" />-->
          </button>
          <button @click="stopRecording" type="button">
            <p>stop</p>
            <!--<img height="30em" src="{stopIcon}" alt="stopRecording" />-->
          </button>
        </div>
        <div v-if="recordedVideo" className="{styles.recordedplayer}">
          <video className="recorded" :src="recordedVideo" controls></video>
          <a download :href="recordedVideo"> Download Recording </a>
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
</style>
