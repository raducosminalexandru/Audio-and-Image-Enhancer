<template>
  <button class="action-btn spectrogram-btn" @click="showSpectrogram" :disabled="!audioSrc">
    <span class="btn-icon">📊</span> View Spectrogram
  </button>
</template>

<script>
export default {
  name: 'SpectrogramButton',
  props: {
    audioSrc: {
      type: String,
      required: true
    }
  },
  methods: {
    showSpectrogram() {
      this.fetchAndProcessAudio(this.audioSrc).then(spectrogramData => {
        this.$emit('spectrogram-generated', spectrogramData);
      }).catch(error => {
        console.error('Error generating spectrogram:', error);
      });
    },

    async fetchAndProcessAudio(url) {
      const audioContext = new (window.AudioContext || window.webkitAudioContext)();

      const response = await fetch(url);
      const arrayBuffer = await response.arrayBuffer();

      const audioBuffer = await audioContext.decodeAudioData(arrayBuffer);

      const rawData = audioBuffer.getChannelData(0);

      const numSlices = Math.min(256, Math.floor(audioBuffer.duration * 10));

      const numBins = 256;

      const spectrogramData = [];

      const samplesPerSlice = Math.floor(rawData.length / numSlices);

      for (let i = 0; i < numSlices; i++) {
        const startSample = i * samplesPerSlice;
        const endSample = Math.min(startSample + samplesPerSlice, rawData.length);

        const sliceData = rawData.slice(startSample, endSample);

        const frequencyData = new Uint8Array(numBins);

        this.simulateFrequencyAnalysis(sliceData, frequencyData);

        spectrogramData.push(Array.from(frequencyData));
      }

      return {
        spectrogramData,
        width: numSlices,
        height: numBins,
        duration: audioBuffer.duration
      };
    },

    simulateFrequencyAnalysis(timeData, frequencyData) {
      const numBins = frequencyData.length;
      const numSamples = timeData.length;

      for (let i = 0; i < numBins; i++) {
        frequencyData[i] = 0;
      }

      for (let bin = 0; bin < numBins; bin++) {
        const binSize = Math.max(1, Math.floor(numSamples / (numBins - bin/2)));
        const binStart = Math.floor((bin / numBins) * (numSamples - binSize));

        let energy = 0;
        for (let j = 0; j < binSize; j++) {
          const idx = binStart + j;
          if (idx < numSamples) {
            energy += Math.abs(timeData[idx]);
          }
        }

        energy = energy / binSize;
        frequencyData[bin] = Math.min(255, Math.max(0, Math.floor(energy * 5000)));
      }

      this.smoothArray(frequencyData);
    },

    smoothArray(array) {
      const len = array.length;
      if (len < 3) return;

      const original = [...array];

      for (let i = 1; i < len - 1; i++) {
        array[i] = Math.floor((original[i-1] + original[i] + original[i+1]) / 3);
      }

      array[0] = original[0];
      array[len - 1] = original[len - 1];
    }
  }
}
</script>

<style scoped>
.spectrogram-btn {
  background: linear-gradient(45deg, #3f51b5, #1a237e);
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



.spectrogram-btn:hover {
  background: linear-gradient(45deg, #303f9f, #0d47a1);
  transform: translateY(-2px);
}

.spectrogram-btn:disabled {
  background: #666;
  cursor: not-allowed;
  opacity: 0.7;
  transform: none;
}
</style>
