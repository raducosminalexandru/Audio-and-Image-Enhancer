<template>
  <div class="frequency-modifier">
    <button
        @click="toggleFrequencyOptions"
        class="action-btn frequency-btn"
    >
      <span class="btn-icon">🔊</span> Modify Speed / Pitch
    </button>

    <div v-if="showOptions" class="frequency-options">
      <button
          v-for="(option, index) in frequencyOptions"
          :key="index"
          @click="applyFrequency(option.value, option.label)"
          class="frequency-option-btn"
      >
        {{ option.label }}
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'FrequencyModifierButton',
  props: {
    mediaType: {
      type: String,
      required: false,
      default: 'audio'
    },
    audioSrc: {
      type: String,
      default: null
    }
  },
  data() {
    return {
      showOptions: false,
      frequencyOptions: [
        { label: '0.5x', value: 0.5 },
        { label: '1.0x', value: 1.0 },
        { label: '2.0x', value: 2.0 }
      ],
      audioContext: null,
      originalAudioBuffer: null
    };
  },
  computed: {
    showFrequencyOptions() {
      return this.mediaType === 'audio' && this.audioSrc;
    }
  },
  methods: {
    toggleFrequencyOptions() {
      this.showOptions = !this.showOptions;
    },
    async applyFrequency(pitchFactor, label) {
      try {
        if (!this.audioContext) {
          this.audioContext = new (window.AudioContext || window.webkitAudioContext)();
        }

        if (!this.originalAudioBuffer) {
          const response = await fetch(this.audioSrc);
          const arrayBuffer = await response.arrayBuffer();
          this.originalAudioBuffer = await this.audioContext.decodeAudioData(arrayBuffer);
        }

        await this.processPitchWithDurationAdjustment(pitchFactor, label);
        this.showOptions = false;
      } catch (error) {
        console.error("Error modifying speed/pitch:", error);
        alert("There was an error modifying the audio. Please try again.");
      }
    },
    async processPitchWithDurationAdjustment(pitchFactor, label) {
      const inputBuffer = this.originalAudioBuffer;
      const originalSampleRate = inputBuffer.sampleRate;
      const originalDuration = inputBuffer.duration;

      const newDuration = originalDuration / pitchFactor;
      const newLength = Math.floor(inputBuffer.length / pitchFactor);

      const offlineContext = new OfflineAudioContext(
          inputBuffer.numberOfChannels,
          newLength,
          originalSampleRate
      );

      const source = offlineContext.createBufferSource();
      source.buffer = inputBuffer;
      source.playbackRate.value = pitchFactor;

      source.connect(offlineContext.destination);
      source.start(0);

      const renderedBuffer = await offlineContext.startRendering();
      const wavBlob = this.audioBufferToWav(renderedBuffer);
      const modifiedAudioUrl = URL.createObjectURL(wavBlob);

      alert(`Your pitch was updated to: ${label}`);
      this.$emit('frequency-modified', modifiedAudioUrl, `${label} (${newDuration.toFixed(2)}s)`);
    },
    audioBufferToWav(buffer) {
      const numOfChannels = buffer.numberOfChannels;
      const sampleRate = buffer.sampleRate;
      const length = buffer.length * numOfChannels * 2;

      const wavBuffer = new ArrayBuffer(44 + length);
      const view = new DataView(wavBuffer);

      this.writeString(view, 0, 'RIFF');
      view.setUint32(4, 36 + length, true);
      this.writeString(view, 8, 'WAVE');

      this.writeString(view, 12, 'fmt ');
      view.setUint32(16, 16, true);
      view.setUint16(20, 1, true);
      view.setUint16(22, numOfChannels, true);
      view.setUint32(24, sampleRate, true);
      view.setUint32(28, sampleRate * numOfChannels * 2, true);
      view.setUint16(32, numOfChannels * 2, true);
      view.setUint16(34, 16, true);

      this.writeString(view, 36, 'data');
      view.setUint32(40, length, true);

      let offset = 44;
      const channelData = [];

      for (let ch = 0; ch < numOfChannels; ch++) {
        channelData.push(buffer.getChannelData(ch));
      }

      for (let i = 0; i < buffer.length; i++) {
        for (let ch = 0; ch < numOfChannels; ch++) {
          let sample = channelData[ch][i];
          sample = Math.max(-1, Math.min(1, sample));
          const int16 = sample < 0 ? sample * 0x8000 : sample * 0x7FFF;
          view.setInt16(offset, int16, true);
          offset += 2;
        }
      }

      return new Blob([wavBuffer], { type: 'audio/wav' });
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
.frequency-modifier {
  position: relative;
  width: auto;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.frequency-btn {
  background: linear-gradient(45deg, #3f51b5, #1a237e);
  margin-bottom: 10px;
  padding: 10px 20px;
  color: white;
  font-weight: bold;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.frequency-btn:hover {
  background: linear-gradient(45deg, #303f9f, #0d47a1);
  transform: translateY(-2px);
}

.frequency-options {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  justify-content: center;
  margin-top: 10px;
  animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(-10px); }
  to { opacity: 1; transform: translateY(0); }
}

.frequency-option-btn {
  padding: 8px 15px;
  border: none;
  border-radius: 6px;
  background: linear-gradient(45deg, #00bcd4, #006064);
  color: white;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
}

.frequency-option-btn:hover {
  background: linear-gradient(45deg, #00acc1, #00838f);
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
}
</style>