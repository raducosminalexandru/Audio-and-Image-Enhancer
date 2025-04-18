<template>
  <div class="frequency-modifier">
    <button
        class="frequency-btn"
        @click="reverseAudio"
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

      // Create audio context
      const audioContext = new (window.AudioContext || window.webkitAudioContext)();
      const request = new XMLHttpRequest();
      request.open('GET', this.audioSrc, true);
      request.responseType = 'arraybuffer';

      request.onload = () => {
        audioContext.decodeAudioData(request.response, (buffer) => {
          // Create a new buffer with reversed data
          const reversedBuffer = audioContext.createBuffer(
              buffer.numberOfChannels,
              buffer.length,
              buffer.sampleRate
          );

          // Reverse each channel
          for (let i = 0; i < buffer.numberOfChannels; i++) {
            const channelData = buffer.getChannelData(i);
            const reversedData = reversedBuffer.getChannelData(i);

            // Reverse the audio data
            for (let j = 0; j < channelData.length; j++) {
              reversedData[j] = channelData[channelData.length - 1 - j];
            }
          }

          // Create a new audio source and connect it to the destination
          const source = audioContext.createBufferSource();
          source.buffer = reversedBuffer;
          source.connect(audioContext.destination);

          // Create a media stream destination
          const destination = audioContext.createMediaStreamDestination();
          source.connect(destination);

          // Create a media recorder to capture the reversed audio
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

          // Start recording and playing
          mediaRecorder.start();
          source.start();

          // Stop after the duration of the audio
          setTimeout(() => {
            source.stop();
            mediaRecorder.stop();
          }, buffer.duration * 1000);
        });
      };

      request.send();
    }
  }
};
</script>

<style scoped>
.frequency-modifier {
  position: relative;
  width: 100%;
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

.btn-icon {
  margin-right: 8px;
}
</style>