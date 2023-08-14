<template>
  <span class="time-value"
    >{{ hours }}:{{ minutes }}:{{ seconds }}:{{ milliseconds }}</span
  >
</template>

<script>
export default {
  name: "Stopwatch",
  props: {
    currentTime: {
      type: Number,
      required: true,
    },
  },
  computed: {
    hours() {
      return String(Math.floor(this.currentTime / 3600000)).padStart(2, "0");
    },
    minutes() {
      return String(Math.floor((this.currentTime / 60000) % 60)).padStart(
        2,
        "0"
      );
    },
    seconds() {
      return String(Math.floor((this.currentTime / 1000) % 60)).padStart(
        2,
        "0"
      );
    },
    milliseconds() {
      return String(this.currentTime % 1000).padStart(3, "0");
    },
  },
};
</script>

<style>
.time-value {
  position: absolute;
  font-size: 1.8em;
  color: white;
  top: 13.5em;
  left: 3.9em;
}
</style>
