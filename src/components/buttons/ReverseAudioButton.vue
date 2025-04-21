<template>
  <div class="reverse-audio">
    <button
        @click="reverseAudio"
        class="action-btn reverse-btn"
    >
      <span class="btn-icon">🔃</span> Reverse Audio
    </button>
  </div>
</template>

<script>
export default {
  name: 'ReverseAudioButton',
  props: {
    audioSrc: {
      type: String,
      required: true
    }
  },
  methods: {
    reverseAudio() {
      if (!this.audioSrc) return;

      const audioContext = new (window.AudioContext || window.webkitAudioContext)();
      const request = new XMLHttpRequest();
      request.open('GET', this.audioSrc, true);
      request.responseType = 'arraybuffer';

      request.onload = () => {
        audioContext.decodeAudioData(request.response, (buffer) => {
          const reversedBuffer = audioContext.createBuffer(
              buffer.numberOfChannels,
              buffer.length,
              buffer.sampleRate
          );

          for (let i = 0; i < buffer.numberOfChannels; i++) {
            const channelData = buffer.getChannelData(i);
            const reversedData = reversedBuffer.getChannelData(i);
            for (let j = 0; j < channelData.length; j++) {
              reversedData[j] = channelData[channelData.length - 1 - j];
            }
          }

          const destination = audioContext.createMediaStreamDestination();
          const source = audioContext.createBufferSource();
          source.buffer = reversedBuffer;
          source.connect(destination);

          const mediaRecorder = new MediaRecorder(destination.stream);
          const audioChunks = [];

          mediaRecorder.ondataavailable = (event) => {
            audioChunks.push(event.data);
          };

          mediaRecorder.onstop = () => {
            const audioBlob = new Blob(audioChunks, { type: 'audio/wav' });
            const audioUrl = URL.createObjectURL(audioBlob);
            this.$emit('audio-reversed', audioUrl);
          };

          mediaRecorder.start();
          source.start();

          setTimeout(() => {
            source.stop();
            mediaRecorder.stop();
            source.disconnect();
          }, buffer.duration * 1000);
        });
      };

      request.send();
    }
  }
};
</script>

<style scoped>
.reverse-audio {
  position: relative;
  width: auto;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.action-btn {
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

.action-btn:hover {
  background: linear-gradient(45deg, #303f9f, #0d47a1);
  transform: translateY(-2px);
}

.reverse-btn .btn-icon {
  margin-right: 8px;
}
</style>
