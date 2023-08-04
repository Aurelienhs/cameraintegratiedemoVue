<template>
  <div class="header">
    <img class="logo" src="@/assets/relivologo.png" alt="logo" />
    Camera integration demo
  </div>
</template>

<script>
export default {
  name: "Banner",
  components: {},
};
</script>

<style>
.header {
  background: linear-gradient(#fe623d, #ff3472);
  height: 50px;
  font-size: 2em;
  color: white;
  padding: 20px;
  text-align: center;
}
.logo {
  width: 50p;
  height: 50px;
  float: left;
}
</style>
