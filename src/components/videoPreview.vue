<template>
  <div class="wrap">
    <video
      :src="url"
      controls
      class="video-container"
      @loadeddata="handleVideoLoaded"
      @error="handleVideoError">
      <p>您的版本过低，无法播放视频文件</p>
    </video>
  </div>
</template>

<script>
export default {
  name: 'video-preview',
  props: {
    url: {
      required: true,
      type: String,
      default: '',
    },
  },
  methods: {
    handleVideoLoaded() {
      this.$emit('loaded', { type: 'video', url: this.url });
    },
    handleVideoError(error) {
      this.$emit('error', { type: 'video', url: this.url, error });
    }
  }
};
</script>
<style scoped>
video{
  max-width: 1000px;
}
</style>
