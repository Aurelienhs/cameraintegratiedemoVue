<template>
  <p>{{ countDown }}</p>
</template>

<script>
export default {
  name: "CountDownTimer",
  data() {
    return {
      countDown: 3,
    };
  },
  methods: {
    countDownTimer() {
      if (this.countDown > 0) {
        setTimeout(() => {
          this.countDown -= 1;
          this.countDownTimer();
        }, 1000);
      } else {
        this.$emit("countdown-complete");
      }
    },
  },
  created() {
    this.countDownTimer();
  },
};
</script>
