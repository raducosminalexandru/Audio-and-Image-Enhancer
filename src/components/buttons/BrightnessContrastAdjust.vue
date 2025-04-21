<template>
  <div class="brightness-contrast-tool">
    <label class="slider-label">Brightness</label>
    <input type="range" min="-100" max="100" v-model="brightness" class="slider" />

    <label class="slider-label">Contrast</label>
    <input type="range" min="-100" max="100" v-model="contrast" class="slider" />

    <button class="custom-btn" @click="applyAdjustments">
      ✨ Apply BC Adjust
    </button>
  </div>
</template>

<script>
export default {
  name: 'BrightnessContrastAdjust',
  props: {
    mediaSrc: String
  },
  data() {
    return {
      brightness: 0,
      contrast: 0,
      originalImageData: null,
      imgWidth: 0,
      imgHeight: 0
    };
  },
  methods: {
    applyAdjustments() {
      if (!this.originalImageData) {
        const img = new Image();
        img.crossOrigin = 'Anonymous';
        img.src = this.mediaSrc;

        img.onload = () => {
          const canvas = document.createElement('canvas');
          const ctx = canvas.getContext('2d');

          canvas.width = img.width;
          canvas.height = img.height;
          ctx.drawImage(img, 0, 0);

          const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
          this.originalImageData = imageData;
          this.imgWidth = canvas.width;
          this.imgHeight = canvas.height;

          this.processAdjustments();
        };
      } else {
        this.processAdjustments();
      }
    },

    processAdjustments() {
      const canvas = document.createElement('canvas');
      const ctx = canvas.getContext('2d');
      canvas.width = this.imgWidth;
      canvas.height = this.imgHeight;

      const imageData = new ImageData(
          new Uint8ClampedArray(this.originalImageData.data),
          this.imgWidth,
          this.imgHeight
      );
      const data = imageData.data;

      const brightnessFactor = this.brightness / 400;
      const contrastFactorRaw = this.contrast / 400;
      const contrastFactor = (259 * (contrastFactorRaw * 255 + 255)) / (255 * (259 - contrastFactorRaw * 255));

      for (let i = 0; i < data.length; i += 4) {
        data[i] += 255 * brightnessFactor;
        data[i + 1] += 255 * brightnessFactor;
        data[i + 2] += 255 * brightnessFactor;

        data[i] = contrastFactor * (data[i] - 128) + 128;
        data[i + 1] = contrastFactor * (data[i + 1] - 128) + 128;
        data[i + 2] = contrastFactor * (data[i + 2] - 128) + 128;
      }

      ctx.putImageData(imageData, 0, 0);
      const newDataUrl = canvas.toDataURL();
      this.$emit('adjusted', newDataUrl);
    }
  }
};
</script>

<style scoped>
.custom-btn {
  background: linear-gradient(45deg, #ec407a, #ad1457);
  padding: 0px 10px;
  height: 24px;
  color: white;
  font-weight: bold;
  font-size: 8px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s ease;
  white-space: nowrap;
  min-width: 110px;
  box-sizing: border-box;
  text-align: center;
  display: inline-flex;
  align-items: center;
  justify-content: center;
}

.custom-btn:hover {
  background: linear-gradient(45deg, #d81b60, #880e4f);
  transform: translateY(-2px);
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(-10px); }
  to { opacity: 1; transform: translateY(0); }
}
</style>

