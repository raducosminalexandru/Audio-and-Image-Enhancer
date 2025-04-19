<template>
  <button
      @click="$emit('remove-media')"
      class="action-btn remove-btn"
      :class="{ 'disabled-btn': disabled }"
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
