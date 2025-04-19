<template>
  <button
      @click="selectMedia"
      class="action-btn upload-btn"
      :class="{ 'disabled-btn': isDisabled }"
      :disabled="isDisabled"
  >
    <span class="btn-icon">📁</span> Select Media
  </button>
</template>

<script>
export default {
  name: 'SelectMediaButton',
  props: {
    selectedFile: {
      type: Object,
      default: null
    }
  },
  computed: {
    isDisabled() {
      return this.selectedFile !== null;
    }
  },
  methods: {
    selectMedia() {
      if (!this.isDisabled) {
        this.$emit('select-media');
      }
    }
  }
}
</script>

<style scoped>
.action-btn {
  padding: 12px 20px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  position: relative;
  overflow: hidden;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
}

.action-btn::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.2), transparent);
  transition: 0.5s;
}

.action-btn:hover::before {
  left: 100%;
}

.btn-icon {
  margin-right: 8px;
  font-size: 1.1rem;
}

.upload-btn {
  background: linear-gradient(45deg, #ffeb3b, #fbc02d);
  color: #333;
}

.disabled-btn {
  opacity: 0.6;
  cursor: not-allowed;
  transform: none;
  pointer-events: none;
}
</style>