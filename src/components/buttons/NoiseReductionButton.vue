<template>
  <button class="action-btn submit-btn noise-reduction-btn" @click="applyNoiseReduction">
    <span class="btn-icon">🔇</span> Noise Reduction
  </button>
</template>

<script>
export default {
  name: 'NoiseReductionButton',
  props: {
    audioSrc: {
      type: String,
      required: true
    }
  },
  methods: {
    applyNoiseReduction() {
      // Create audio context
      const audioContext = new (window.AudioContext || window.webkitAudioContext)();

      // Fetch the audio data
      fetch(this.audioSrc)
          .then(response => response.arrayBuffer())
          .then(arrayBuffer => audioContext.decodeAudioData(arrayBuffer))
          .then(audioBuffer => {
            // Get audio data
            const channelData = audioBuffer.getChannelData(0); // Get first channel

            // Simple noise reduction algorithm
            // 1. Detect noise floor (assuming initial portion contains background noise)
            const sampleSize = Math.min(5000, channelData.length / 10); // Sample first 5000 samples or 10% of file
            let sum = 0;

            for (let i = 0; i < sampleSize; i++) {
              sum += Math.abs(channelData[i]);
            }

            // Calculate average noise amplitude
            const noiseThreshold = sum / sampleSize * 2; // Threshold slightly above noise floor

            // Create a new buffer for processed audio
            const processedBuffer = audioContext.createBuffer(
                audioBuffer.numberOfChannels,
                audioBuffer.length,
                audioBuffer.sampleRate
            );

            // Process each channel
            for (let channel = 0; channel < audioBuffer.numberOfChannels; channel++) {
              const inputData = audioBuffer.getChannelData(channel);
              const outputData = processedBuffer.getChannelData(channel);

              // Apply noise gate - if sample is below threshold, reduce it significantly
              for (let i = 0; i < inputData.length; i++) {
                if (Math.abs(inputData[i]) < noiseThreshold) {
                  outputData[i] = inputData[i] * 0.1; // Reduce noise by 90%
                } else {
                  outputData[i] = inputData[i]; // Keep signal above threshold
                }
              }
            }

            // Convert the processed buffer back to an audio element source
            const audioBlob = this.bufferToWave(processedBuffer, processedBuffer.length);
            const audioUrl = URL.createObjectURL(audioBlob);

            // Emit event with the new audio URL
            this.$emit('noise-reduced', audioUrl);
          })
          .catch(error => {
            console.error('Error processing audio:', error);
            alert('Error applying noise reduction');
          });
    },

    // Convert AudioBuffer to WAV format blob
    bufferToWave(audioBuffer, len) {
      const numOfChannels = audioBuffer.numberOfChannels;
      const sampleRate = audioBuffer.sampleRate;
      const format = 1; // PCM
      const bitDepth = 16;

      const bytesPerSample = bitDepth / 8;
      const blockAlign = numOfChannels * bytesPerSample;

      const buffer = new ArrayBuffer(44 + len * blockAlign);
      const view = new DataView(buffer);

      // Write WAVE header
      this.writeString(view, 0, 'RIFF');
      view.setUint32(4, 36 + len * blockAlign, true);
      this.writeString(view, 8, 'WAVE');
      this.writeString(view, 12, 'fmt ');
      view.setUint32(16, 16, true);
      view.setUint16(20, format, true);
      view.setUint16(22, numOfChannels, true);
      view.setUint32(24, sampleRate, true);
      view.setUint32(28, sampleRate * blockAlign, true);
      view.setUint16(32, blockAlign, true);
      view.setUint16(34, bitDepth, true);
      this.writeString(view, 36, 'data');
      view.setUint32(40, len * blockAlign, true);

      // Write interleaved data
      const offset = 44;
      const channels = [];

      // Extract channels
      for (let i = 0; i < numOfChannels; i++) {
        channels.push(audioBuffer.getChannelData(i));
      }

      // Write samples
      for (let i = 0; i < len; i++) {
        for (let channel = 0; channel < numOfChannels; channel++) {
          // Fixed: Use let instead of const for sample variable
          let sample = Math.max(-1, Math.min(1, channels[channel][i]));
          sample = sample < 0 ? sample * 0x8000 : sample * 0x7FFF;
          view.setInt16(offset + (i * blockAlign) + (channel * bytesPerSample), sample, true);
        }
      }

      return new Blob([buffer], { type: 'audio/wav' });
    },

    writeString(view, offset, string) {
      for (let i = 0; i < string.length; i++) {
        view.setUint8(offset + i, string.charCodeAt(i));
      }
    }
  }
};
</script>

<style scoped>
.noise-reduction-btn {
  background: linear-gradient(45deg, #8e44ad, #673ab7);
}

.noise-reduction-btn:hover {
  background: linear-gradient(45deg, #7e349d, #5e35b1);
  transform: translateY(-2px);
}
</style>