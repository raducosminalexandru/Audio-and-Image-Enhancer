<template>
  <div class="spectrogram-container" v-if="isVisible">
    <div class="spectrogram-header">
      <h3 :class="{ 'light-text': theme === 'light' }">Audio Spectrogram</h3>
      <button class="close-btn" @click="close">×</button>
    </div>

    <div class="spectrogram-content">
      <canvas ref="spectrogramCanvas" class="spectrogram-canvas"></canvas>
    </div>

    <div class="spectrogram-legend">
      <div class="legend-gradient"></div>
      <div class="legend-labels">
        <span>Low</span>
        <span>Amplitude</span>
        <span>High</span>
      </div>
    </div>

    <div class="spectrogram-info">
      <p :class="{ 'light-text': theme === 'light' }">
        Frequency (vertical) vs Time (horizontal)
      </p>
    </div>
  </div>
</template>

<script>
export default {
  name: 'SpectrogramDisplay',
  props: {
    spectrogramData: {
      type: Object,
      required: true
    },
    isVisible: {
      type: Boolean,
      default: false
    },
    theme: {
      type: String,
      default: 'dark'
    }
  },
  watch: {
    spectrogramData: {
      handler(newData) {
        if (newData && this.isVisible) {
          this.$nextTick(() => {
            this.drawSpectrogram();
          });
        }
      },
      deep: true
    },
  },
  mounted() {
    if (this.isVisible && this.spectrogramData) {
      this.drawSpectrogram();
    }
  },
  methods: {
    drawSpectrogram() {
      const canvas = this.$refs.spectrogramCanvas;
      if (!canvas || !this.spectrogramData) return;

      const ctx = canvas.getContext('2d');
      const { spectrogramData, width, height } = this.spectrogramData;

      canvas.width = width;
      canvas.height = height;

      ctx.clearRect(0, 0, canvas.width, canvas.height);

      const colorGradient = this.generateColorGradient();

      for (let x = 0; x < spectrogramData.length; x++) {
        const column = spectrogramData[x];

        for (let y = 0; y < column.length; y++) {
          const invertedY = height - y - 1;

          const amplitude = column[y];
          const color = colorGradient[Math.floor((amplitude / 255) * (colorGradient.length - 1))];

          ctx.fillStyle = color;
          ctx.fillRect(x, invertedY, 1, 1);
        }
      }
    },

    generateColorGradient() {
      const colors = [];

      for (let i = 0; i < 64; i++) {
        colors.push(`rgb(0, 0, ${Math.floor((i / 64) * 255)})`);
      }

      for (let i = 0; i < 64; i++) {
        colors.push(`rgb(${Math.floor((i / 64) * 255)}, 0, 255)`);
      }

      for (let i = 0; i < 64; i++) {
        colors.push(`rgb(255, 0, ${255 - Math.floor((i / 64) * 255)})`);
      }

      for (let i = 0; i < 64; i++) {
        colors.push(`rgb(255, ${Math.floor((i / 64) * 255)}, 0)`);
      }

      return colors;
    },

    close() {
      this.$emit('close');
    }
  }
}
</script>

<style scoped>
.spectrogram-container {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 80%;
  max-width: 800px;
  background-color: #1e1e1e;
  border-radius: 12px;
  box-shadow: 0 0 30px rgba(0, 0, 0, 0.5);
  padding: 20px;
  z-index: 1000;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.light-theme .spectrogram-container {
  background-color: #ffffff;
  box-shadow: 0 0 30px rgba(0, 0, 0, 0.2);
}

.spectrogram-header {
  width: 100%;
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.spectrogram-header h3 {
  margin: 0;
}

.close-btn {
  background: none;
  border: none;
  color: #fff;
  font-size: 24px;
  cursor: pointer;
  transition: color 0.3s ease;
}

.light-theme .close-btn {
  color: #333;
}

.close-btn:hover {
  color: #ff6ec4;
}

.spectrogram-content {
  width: 100%;
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
  overflow: hidden;
}

.spectrogram-canvas {
  width: 100%;
  height: 300px;
  image-rendering: pixelated;
  border: 1px solid #444;
  border-radius: 4px;
}

.light-theme .spectrogram-canvas {
  border: 1px solid #ddd;
}

.spectrogram-legend {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 10px;
}

.legend-gradient {
  width: 80%;
  height: 20px;
  margin-bottom: 5px;
  background: linear-gradient(to right,
  #000033, #0000ff, #ff00ff, #ff0000, #ffff00);
  border-radius: 4px;
}

.legend-labels {
  width: 80%;
  display: flex;
  justify-content: space-between;
  font-size: 0.8rem;
  color: #ccc;
}

.light-theme .legend-labels {
  color: #666;
}

.spectrogram-info {
  width: 100%;
  text-align: center;
  font-size: 0.9rem;
  color: #ccc;
}

.light-theme .spectrogram-info {
  color: #666;
}
</style>