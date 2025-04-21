<template>
  <div :class="{ 'light-theme': theme === 'light', 'dark-theme': theme === 'dark' }">
    <div class="top-bar" :class="{ 'light-top-bar': theme === 'light', 'dark-top-bar': theme === 'dark' }">
      <ThemeToggle :theme="theme" :toggleTheme="toggleTheme" />
    </div>

    <div class="app-background">
      <div class="app-header">
        <div class="title-container">
          <h1 class="main-title" :class="{ 'light-title': theme === 'light' }">
            <span class="audio-icon">🎧</span> Audio and <span class="image-icon">🎨</span> Image Enhancer
          </h1>
        </div>
        <div class="subtitle-container">
          <p class="subtitle" :class="{ 'light-subtitle': theme === 'light' }">
            Upload your media and enhance it with a click
          </p>
        </div>
      </div>

      <div class="file-upload" :class="{ 'light-panel': theme === 'light' }">
        <div class="upload-container">
          <input
              type="file"
              @change="handleFileUpload"
              accept=".jpg,.jpeg,.png,.wav,.mp3"
              ref="fileInput"
              class="hidden-input"
          />
          <div class="buttons-row">
            <SelectMediaButton
                :selectedFile="selectedFile"
                @select-media="$refs.fileInput.click()"
            />
            <RemoveMediaButton
                :disabled="!selectedFile"
                @remove-media="removeMedia"
            />
          </div>
          <span v-if="selectedFile" class="file-name" :class="{ 'light-text': theme === 'light' }">
            {{ selectedFile.name }}
          </span>
        </div>

        <div v-if="mediaPreview" class="preview-container">
          <h3 :class="{ 'light-text': theme === 'light' }">Preview</h3>

          <img
              v-if="mediaType === 'image'"
              :src="mediaPreview"
              alt="Image Preview"
              class="media-preview"
              :class="{ 'light-border': theme === 'light' }"
              ref="imageElement"
          />

          <div v-if="mediaType === 'image'" class="buttons-row">
            <button class="action-btn submit-btn" @click="restoreOriginal">
              <span class="btn-icon">🔁</span> Original
            </button>
            <button class="action-btn submit-btn" @click="applyGrayscale">
              <span class="btn-icon">🎞️</span> Grayscale
            </button>
            <button class="action-btn submit-btn" @click="applyLowExposure">
              <span class="btn-icon">🌑</span> Low Exposure
            </button>
          </div>

          <audio
              v-if="mediaType === 'audio'"
              controls
              class="audio-preview"
              :class="{ 'light-audio': theme === 'light' }"
              ref="audioElement"
          >
            <source :src="mediaPreview" :type="selectedFile.type">
            Your browser does not support the audio element.
          </audio>

          <div v-if="mediaType === 'audio'" class="buttons-row audio-tools">
            <FrequencyModifierButton
                :audioSrc="mediaPreview"
                @frequency-modified="handleFrequencyModified"
            />
            <ReverseAudioButton
                :audioSrc="mediaPreview"
                @audio-reversed="handleAudioReversed"
            />
            <EqualizerButton
                :audioSrc="mediaPreview"
                @equalizer-applied="handleEqualizerApplied"
            />
          </div>

          <div class="upload-download-row">
            <button
                v-if="selectedFile"
                @click="downloadMedia"
                class="action-btn download-btn"
            >
              <span class="btn-icon">⬇️</span> Download Media
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import ThemeToggle from './buttons/ThemeToggleButton.vue';
import FrequencyModifierButton from './buttons/FrequencyModifierButton.vue';
import ReverseAudioButton from './buttons/ReverseAudioButton.vue';
import SelectMediaButton from './buttons/SelectMediaButton.vue';
import RemoveMediaButton from './buttons/RemoveMediaButton.vue';
import EqualizerButton from './buttons/EqualizerButton.vue';

export default {
  name: 'MediaUpload',
  components: {
    ThemeToggle,
    FrequencyModifierButton,
    ReverseAudioButton,
    SelectMediaButton,
    RemoveMediaButton,
    EqualizerButton
  },
  data() {
    return {
      selectedFile: null,
      mediaPreview: null,
      mediaType: null,
      theme: 'dark',
      originalImageData: null
    };
  },
  mounted() {
    // Set initial theme on body and html
    document.body.className = this.theme + '-theme';
    document.documentElement.className = this.theme + '-theme';
  },
  methods: {
    toggleTheme() {
      this.theme = this.theme === 'dark' ? 'light' : 'dark';
      // Apply theme to document elements
      document.body.className = this.theme + '-theme';
      document.documentElement.className = this.theme + '-theme';
    },
    handleFileUpload(event) {
      const file = event.target.files[0];
      if (!file) return;

      const validImageTypes = ['image/jpeg', 'image/png'];
      const validAudioTypes = ['audio/wav', 'audio/mp3', 'audio/mpeg'];

      if (validImageTypes.includes(file.type)) {
        this.mediaType = 'image';
      } else if (validAudioTypes.includes(file.type)) {
        this.mediaType = 'audio';
      } else {
        alert('Please select a JPEG/PNG image or WAV/MP3 audio file.');
        return;
      }

      this.selectedFile = file;

      const reader = new FileReader();
      reader.onload = (e) => {
        this.mediaPreview = e.target.result;
        if (this.mediaType === 'image') {
          this.originalImageData = e.target.result;
        }
      };
      reader.readAsDataURL(file);
    },
    removeMedia() {
      this.selectedFile = null;
      this.mediaPreview = null;
      this.mediaType = null;
      this.originalImageData = null;
      this.$refs.fileInput.value = '';
    },
    uploadFile() {
      if (!this.selectedFile) return;
      alert(`File "${this.selectedFile.name}" ready for upload!`);
    },
    downloadMedia() {
      if (!this.mediaPreview || !this.selectedFile) return;

      const link = document.createElement('a');
      link.href = this.mediaPreview;
      link.download = this.selectedFile.name || 'media';
      link.click();
    },
    applyGrayscale() {
      this.processImage((ctx, canvas) => {
        const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
        const data = imageData.data;

        for (let i = 0; i < data.length; i += 4) {
          const avg = (data[i] + data[i + 1] + data[i + 2]) / 3;
          data[i] = data[i + 1] = data[i + 2] = avg;
        }

        ctx.putImageData(imageData, 0, 0);
      });
    },
    applyLowExposure() {
      this.processImage((ctx, canvas) => {
        const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
        const data = imageData.data;

        for (let i = 0; i < data.length; i += 4) {
          data[i] *= 0.6;
          data[i + 1] *= 0.6;
          data[i + 2] *= 0.6;
        }

        ctx.putImageData(imageData, 0, 0);
      });
    },
    restoreOriginal() {
      this.mediaPreview = this.originalImageData;
    },
    processImage(callback) {
      const img = new Image();
      img.crossOrigin = 'Anonymous';
      img.src = this.mediaPreview;

      img.onload = () => {
        const canvas = document.createElement('canvas');
        canvas.width = img.width;
        canvas.height = img.height;

        const ctx = canvas.getContext('2d');
        ctx.drawImage(img, 0, 0);

        callback(ctx, canvas);

        this.mediaPreview = canvas.toDataURL();
      };
    },
    handleFrequencyModified(newAudioUrl) {
      this.mediaPreview = newAudioUrl;
      if (this.$refs.audioElement) {
        this.$refs.audioElement.load();
      }
    },
    handleAudioReversed(newAudioUrl) {
      this.mediaPreview = newAudioUrl;
      if (this.$refs.audioElement) {
        this.$refs.audioElement.load();
      }
    },
    handleEqualizerApplied(newAudioUrl) {
      this.mediaPreview = newAudioUrl;
      if (this.$refs.audioElement) {
        this.$refs.audioElement.load();
      }
    }
  }
};
</script>

<style>
html, body {
  min-height: 100%;
  height: 100%;
  margin: 0;
  padding: 0;
}

body.dark-theme,
html.dark-theme {
  background-color: #000;
}

body.light-theme,
html.light-theme {
  background-color: #f5f5f7;
}

body {
  margin: 0;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  transition: background-color 0.3s ease, color 0.3s ease;
}

.dark-theme {
  background-color: #000;
  color: #f0f0f0;
}

.light-theme {
  background-color: #f5f5f7;
  color: #333;
}

.top-bar {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  height: 60px;
  display: flex;
  justify-content: flex-end;
  align-items: center;
  padding: 0 20px;
  z-index: 100;
  transition: background-color 0.3s ease;
}

.dark-top-bar {
  background-color: #121212;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
}

.light-top-bar {
  background-color: #ffffff;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.app-background {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 40px 20px;
  min-height: 100vh;
  padding-top: 80px;
  transition: background-color 0.3s ease;
  background-color: inherit;
  width: 100%;
  box-sizing: border-box;
}

.app-header {
  text-align: center;
  margin-bottom: 40px;
}

.title-container {
  margin-top: -20px;
}

.main-title {
  font-size: 3rem;
  font-weight: bold;
  margin-bottom: 10px;
  background: linear-gradient(90deg, #00e5ff, #ff6ec4);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.audio-icon,
.image-icon {
  font-size: 2rem;
  vertical-align: middle;
}

.light-title {
  background: linear-gradient(90deg, #0077cc, #cc00cc);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.subtitle-container {
  margin-top: 20px;
}

.subtitle {
  font-size: 1.1rem;
  color: #ccc;
  font-style: italic;
  letter-spacing: 0.5px;
  text-shadow: 0 0 5px rgba(255, 255, 255, 0.1);
  max-width: 600px;
  margin: 0 auto;
}

.light-subtitle {
  color: #666;
  text-shadow: none;
}

.file-upload {
  background-color: #1e1e1e;
  padding: 30px;
  border-radius: 12px;
  box-shadow: 0 4px 15px rgba(255, 255, 255, 0.1);
  max-width: 500px;
  width: 100%;
  text-align: center;
  transition: background-color 0.3s ease, box-shadow 0.3s ease;
}

.light-panel {
  background-color: #ffffff;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
}

h3 {
  margin-bottom: 20px;
  color: #fff;
  transition: color 0.3s ease;
}

.light-text {
  color: #333;
}

.hidden-input {
  display: none;
}

.upload-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 20px;
}

.buttons-row {
  display: flex;
  flex-wrap: wrap;
  gap: 15px;
  justify-content: center;
  margin-bottom: 15px;
}

.audio-tools {
  display: flex;
  flex-direction: row;
  justify-content: center;
  gap: 15px;
  flex-wrap: nowrap;
  margin-top: 15px;
  width: 100%;
}

.upload-download-row {
  display: flex;
  flex-wrap: wrap;
  gap: 15px;
  justify-content: center;
  margin-top: 25px;
}

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

.file-name {
  font-size: 0.95rem;
  color: #ccc;
  margin-top: 5px;
  word-break: break-word;
  max-width: 100%;
  transition: color 0.3s ease;
}

.preview-container {
  margin-top: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.media-preview {
  max-width: 100%;
  max-height: 300px;
  border-radius: 8px;
  border: 1px solid #444;
  margin-bottom: 15px;
  transition: border 0.3s ease;
}

.light-border {
  border: 1px solid #ddd;
}

.audio-preview {
  width: 100%;
  max-width: 400px;
  margin-bottom: 15px;
  border-radius: 8px;
  background-color: #222;
  padding: 10px;
  transition: background-color 0.3s ease;
}

.light-audio {
  background-color: #f0f0f0;
}

.submit-btn {
  background: linear-gradient(45deg, #ec407a, #ad1457);
}

.submit-btn:hover {
  background: linear-gradient(45deg, #d81b60, #880e4f);
  transform: translateY(-2px);
}

.download-btn {
  background: linear-gradient(45deg, #4caf50, #2e7d32);
}

.download-btn:hover {
  background: linear-gradient(45deg, #43a047, #1b5e20);
}
</style>