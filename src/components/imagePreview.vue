<template>
  <div class="wrap">
    <img :src="url" class="image-container" alt="图片" @load="handleImageLoaded" @error="handleImageError" />
  </div>
</template>

<script>
export default {
  name: 'image-preview',
  props: {
    url: {
      required: true,
      type: String,
      default: '',
    },
  },
  methods: {
    handleImageLoaded() {
      this.$emit('loaded', { type: 'image', url: this.url });
    },
    handleImageError(error) {
      this.$emit('error', { type: 'image', url: this.url, error });
    }
  }
};
</script>
<style scoped>
  img{
    max-width: 1000px;
  }
</style>
