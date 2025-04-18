<template>
  <button @click="uploadFile" class="action-btn upload-media-btn">
    <span class="btn-icon">⬆️</span> Upload Media
  </button>
</template>

<script>
export default {
  name: 'UploadMediaButton',
  props: {
    selectedFile: Object,
  },
  methods: {
    uploadFile() {
      if (!this.selectedFile) return;
      alert(`File "${this.selectedFile.name}" ready for upload!`);
    }
  }
};
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

.upload-media-btn {
  background: linear-gradient(45deg, #2196F3, #1565C0);
  color: white;
}

.upload-media-btn:hover {
  background: linear-gradient(45deg, #1E88E5, #0D47A1);
}
</style>
