<template>
  <div class="equalizer-container">
    <button class="action-btn equalizer-btn" @click="toggleEqualizer">
      <span class="btn-icon">🎚️</span> Equalizer
    </button>

    <div v-if="showEqualizer" class="equalizer-panel">
      <div class="freq-sliders">
        <div v-for="(band, index) in bands" :key="index" class="freq-band">
          <span class="band-label">{{ band.label }}</span>
          <input
              type="range"
              min="-12"
              max="12"
              step="1"
              v-model="band.gain"
              class="eq-slider"
              @input="updateEqualizer"
          />
          <span class="gain-value">{{ band.gain }}dB</span>
        </div>
      </div>
      <div class="eq-buttons">
        <button class="eq-preset-btn" @click="applyPreset('flat')">Flat</button>
        <button class="eq-preset-btn" @click="applyPreset('bass')">Bass Boost</button>
        <button class="eq-preset-btn" @click="applyPreset('vocal')">Vocal</button>
      </div>
      <button class="eq-apply-btn" @click="applyEqualizer">Apply EQ</button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'EqualizerButton',
  props: {
    audioSrc: {
      type: String,
      default: null
    }
  },
  data() {
    return {
      showEqualizer: false,
      audioContext: null,
      audioElement: null,
      source: null,
      filters: [],
      bands: [
        { freq: 60, gain: 0, label: '60Hz' },
        { freq: 170, gain: 0, label: '170Hz' },
        { freq: 310, gain: 0, label: '310Hz' },
        { freq: 600, gain: 0, label: '600Hz' },
        { freq: 1000, gain: 0, label: '1kHz' },
        { freq: 3000, gain: 0, label: '3kHz' },
        { freq: 6000, gain: 0, label: '6kHz' },
        { freq: 12000, gain: 0, label: '12kHz' },
      ]
    };
  },
  methods: {
    toggleEqualizer() {
      this.showEqualizer = !this.showEqualizer;
      if (this.showEqualizer && this.audioSrc && !this.audioContext) {
        this.initAudio();
      }
    },
    initAudio() {
      try {
        this.audioContext = new (window.AudioContext || window.webkitAudioContext)();
        this.audioElement = new Audio(this.audioSrc);
        this.source = this.audioContext.createMediaElementSource(this.audioElement);

        this.filters = this.bands.map(band => {
          const filter = this.audioContext.createBiquadFilter();
          filter.type = band.freq < 80 ? 'lowshelf' :
              band.freq > 10000 ? 'highshelf' : 'peaking';
          filter.frequency.value = band.freq;
          filter.gain.value = band.gain;
          filter.Q.value = 1;
          return filter;
        });

        this.source.connect(this.filters[0]);
        for (let i = 0; i < this.filters.length - 1; i++) {
          this.filters[i].connect(this.filters[i + 1]);
        }
        this.filters[this.filters.length - 1].connect(this.audioContext.destination);
      } catch (error) {
        console.error('Error initializing audio context:', error);
      }
    },
    updateEqualizer() {
      if (!this.audioContext || !this.filters.length) return;
      this.filters.forEach((filter, index) => {
        filter.gain.value = Number(this.bands[index].gain);
      });
    },
    applyPreset(type) {
      switch(type) {
        case 'flat':
          this.bands.forEach(band => { band.gain = 0; });
          break;
        case 'bass':
          this.bands[0].gain = 10;
          this.bands[1].gain = 8;
          this.bands[2].gain = 4;
          this.bands[3].gain = 2;
          this.bands[4].gain = 0;
          this.bands[5].gain = 0;
          this.bands[6].gain = 1;
          this.bands[7].gain = 2;
          break;
        case 'vocal':
          this.bands[0].gain = -2;
          this.bands[1].gain = -1;
          this.bands[2].gain = 0;
          this.bands[3].gain = 3;
          this.bands[4].gain = 4;
          this.bands[5].gain = 3;
          this.bands[6].gain = 1;
          this.bands[7].gain = 0;
          break;
      }
      this.updateEqualizer();
    },
    applyEqualizer() {
      if (!this.audioSrc) return;

      this.processAudio().then(processedBuffer => {
        const wavBlob = this.bufferToWave(processedBuffer, processedBuffer.length);
        const processedUrl = URL.createObjectURL(wavBlob);
        this.$emit('equalizer-applied', processedUrl);
        this.showEqualizer = false;
      });
    },
    async processAudio() {
      const response = await fetch(this.audioSrc);
      const arrayBuffer = await response.arrayBuffer();

      const tempContext = new (window.AudioContext || window.webkitAudioContext)();
      const decodedBuffer = await tempContext.decodeAudioData(arrayBuffer);
      const { numberOfChannels, length, sampleRate } = decodedBuffer;

      const offlineContext = new OfflineAudioContext({
        numberOfChannels,
        length,
        sampleRate
      });

      const source = offlineContext.createBufferSource();
      source.buffer = decodedBuffer;

      const filters = this.bands.map(band => {
        const filter = offlineContext.createBiquadFilter();
        filter.type = band.freq < 80 ? 'lowshelf' :
            band.freq > 10000 ? 'highshelf' : 'peaking';
        filter.frequency.value = band.freq;
        filter.gain.value = band.gain;
        filter.Q.value = 1;
        return filter;
      });

      source.connect(filters[0]);
      for (let i = 0; i < filters.length - 1; i++) {
        filters[i].connect(filters[i + 1]);
      }
      filters[filters.length - 1].connect(offlineContext.destination);

      source.start(0);
      return await offlineContext.startRendering();
    },
    bufferToWave(abuffer, len) {
      const numOfChan = abuffer.numberOfChannels;
      const length = len * numOfChan * 2 + 44;
      const buffer = new ArrayBuffer(length);
      const view = new DataView(buffer);
      const channels = [];

      this.writeString(view, 0, 'RIFF');
      view.setUint32(4, length - 8, true);
      this.writeString(view, 8, 'WAVE');
      this.writeString(view, 12, 'fmt ');
      view.setUint32(16, 16, true);
      view.setUint16(20, 1, true);
      view.setUint16(22, numOfChan, true);
      view.setUint32(24, abuffer.sampleRate, true);
      view.setUint32(28, abuffer.sampleRate * 2 * numOfChan, true);
      view.setUint16(32, numOfChan * 2, true);
      view.setUint16(34, 16, true);
      this.writeString(view, 36, 'data');
      view.setUint32(40, length - 44, true);

      let offset = 44;
      for (let i = 0; i < abuffer.numberOfChannels; i++) {
        channels.push(abuffer.getChannelData(i));
      }

      for (let i = 0; i < abuffer.length; i++) {
        for (let ch = 0; ch < numOfChan; ch++) {
          const sample = Math.max(-1, Math.min(1, channels[ch][i]));
          const int16 = sample < 0 ? sample * 0x8000 : sample * 0x7FFF;
          view.setInt16(offset, int16, true);
          offset += 2;
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
.equalizer-container {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.equalizer-btn {
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

.equalizer-btn:hover {
  background: linear-gradient(45deg, #303f9f, #0d47a1);
  transform: translateY(-2px);
}

.equalizer-panel {
  position: absolute;
  top: 100%;
  left: 50%;
  transform: translateX(-50%);
  width: 300px;
  background: #222;
  border-radius: 8px;
  padding: 15px;
  margin-top: 10px;
  z-index: 10;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.5);
  animation: fadeIn 0.3s ease;
}

.light-theme .equalizer-panel {
  background: #f0f0f0;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
}

.freq-sliders {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-bottom: 15px;
}

.freq-band {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.band-label {
  width: 40px;
  font-size: 0.8rem;
  color: #ddd;
  text-align: right;
  margin-right: 8px;
}

.light-theme .band-label, .light-theme .gain-value {
  color: #444;
}

.eq-slider {
  flex: 1;
  -webkit-appearance: none;
  height: 8px;
  border-radius: 4px;
  background: #444;
  outline: none;
}

.eq-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 15px;
  height: 15px;
  border-radius: 50%;
  background: #8e24aa;
  cursor: pointer;
}

.gain-value {
  width: 40px;
  font-size: 0.8rem;
  color: #ddd;
  text-align: left;
  margin-left: 8px;
}

.eq-buttons {
  display: flex;
  justify-content: space-between;
  margin-bottom: 15px;
}

.eq-preset-btn {
  padding: 6px 12px;
  border: none;
  border-radius: 4px;
  background: #444;
  color: white;
  font-size: 0.85rem;
  cursor: pointer;
  transition: all 0.2s ease;
}

.eq-preset-btn:hover {
  background: #555;
}

.eq-apply-btn {
  width: 100%;
  padding: 8px;
  border: none;
  border-radius: 6px;
  background: linear-gradient(45deg, #8e24aa, #4a148c);
  color: white;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
}

.eq-apply-btn:hover {
  background: linear-gradient(45deg, #7b1fa2, #38006b);
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translate(-50%, -10px);
  }
  to {
    opacity: 1;
    transform: translate(-50%, 0);
  }
}

.light-theme .eq-slider {
  background: #ccc;
}

.light-theme .eq-slider::-webkit-slider-thumb {
  background: #7b1fa2;
}

.light-theme .eq-preset-btn {
  background: #ddd;
  color: #333;
}

.light-theme .eq-preset-btn:hover {
  background: #ccc;
}
</style>