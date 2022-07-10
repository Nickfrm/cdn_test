<template>
  <div class="home">
    <div class="wrap">
      <h1>Simple Video Player Example</h1>
      <VideoPlayer msg="Welcome to Your Vue.js App" />
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
import VideoPlayer from "@/components/VideoPlayer.vue";

export default {
  name: "HomeView",
  components: {
    VideoPlayer,
  },
};
</script>
<style scoped>
.wrap {
  width: 100%;
  max-width: 1280px;
  margin: 0 auto;
}
</style>
