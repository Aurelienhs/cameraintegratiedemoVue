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
            @click="changeRecordingStatus('recording')"
            type="button"
            className="{styles.recordbutton}"
          >
            <img
              height="30em"
              position="absolute"
              src="{playIcon}"
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
          <p className="{styles.timer}">{seconds}</p>
          <button
            className="{styles.pausebutton}"
            onClick="{pauseRecording}"
            type="button"
          >
            <img height="30em" src="{pauseIcon}" alt="pauseRecording" />
          </button>
          <button
            className="{styles.recordbutton}"
            onClick="{stopRecording}"
            type="button"
          >
            <img height="30em" src="{stopIcon}" alt="stopRecording" />
          </button>
        </div>
        <div v-if="recordedVideo" className="{styles.recordedplayer}">
          <video className="recorded" src="{recordedVideo}" controls></video>
          <a
            className="{styles.opencamerabutton}"
            download
            href="{recordedVideo}"
          >
            Download Recording
          </a>
        </div>
      </div>
    </main>
  </div>
</template>

<script>
export default {
  name: "Recorder",
  components: {},
  data() {
    return {
      permission: false,
      mediaRecorder: null,
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

          console.log("youve reached the end");
        } catch (err) {
          alert(err.message);
        }
      } else {
        alert("The MediaRecorder API is not supported in your browser.");
      }
    },
    beforeDestroy() {
      if (this.videoStream) {
        this.videoStream.getTracks().forEach((track) => track.stop());
      }
    },

    startRecording: async () => {
      var mimeType = "video/webm";
      setRecordingStatus("recording");
      if (!mediaRecorder.current) {
        const media = new MediaRecorder(stream, { mimeType });
        mediaRecorder.current = media;
      }
      if (mediaRecorder.current.state === "paused") {
        mediaRecorder.current.resume();
      } else {
        mediaRecorder.current.start();
      }
      start();
      let localVideoChunks = [];
      mediaRecorder.current.ondataavailable = (event) => {
        if (typeof event.data === "undefined") return;
        if (event.data.size === 0) return;
        localVideoChunks.push(event.data);
      };
      setVideoChunks(localVideoChunks);
    },

    pauseRecording: async () => {
      setRecordingStatus("inactive");
      mediaRecorder.current.pause();
      pause();
    },

    stopRecording: () => {
      stop();
      setPermission(false);
      setRecordingStatus("inactive");
      mediaRecorder.current.stop();
      mediaRecorder.current.onstop = () => {
        const videoBlob = new Blob(videoChunks, { type: mimeType });
        const videoUrl = URL.createObjectURL(videoBlob);
        setRecordedVideo(videoUrl);
        setVideoChunks([]);
      };
    },
  },
};
</script>

<style>
main {
  position: relative;
}
</style>
