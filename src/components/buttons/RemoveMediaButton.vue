<template>
  <button
      @click="$emit('remove-media')"
      class="action-btn remove-btn"
      :class="{ 'disabled-btn': disabled }"
      :disabled="disabled"
  >
    <span class="btn-icon">🗑️</span> Remove Media
  </button>
</template>

<script>
export default {
  name: 'RemoveMediaButton',
  props: {
    disabled: {
      type: Boolean,
      default: false
    }
  }
};
</script>

<style scoped>
.remove-btn {
  background: linear-gradient(45deg, #F44336, #C62828);
}

.disabled-btn {
  opacity: 0.6;
  cursor: not-allowed;
  transform: none;
}
</style>
