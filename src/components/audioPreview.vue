<template>
  <div class="wrap">
    <audio
      :src="url"
      class="audio-container"
      controls
      @loadeddata="handleAudioLoaded"
      @error="handleAudioError">
      <p>您的版本过低，无法播放音频文件</p>
    </audio>
  </div>
</template>

<script>
export default {
  name: 'audio-preview',
  props: {
    url: {
      required: true,
      type: String,
      default: '',
    },
  },
  methods: {
    handleAudioLoaded() {
      this.$emit('loaded', { type: 'audio', url: this.url });
    },
    handleAudioError(error) {
      this.$emit('error', { type: 'audio', url: this.url, error });
    }
  }
};
</script>

