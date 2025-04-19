<template>
  <div class="frequency-modifier">
    <button class="frequency-btn" @click="reverseAudio">
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

          // Create a media stream destination to capture the audio
          const destination = audioContext.createMediaStreamDestination();

          // Create a temporary source to connect to the destination (but we won't play it)
          const source = audioContext.createBufferSource();
          source.buffer = reversedBuffer;
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

            // Emit the reversed audio URL to update the preview without playing
            this.$emit('audio-reversed', audioUrl);
          };

          // Start recording but don't play the audio
          mediaRecorder.start();

          // Connect to audio processing graph but don't connect to audioContext.destination
          // This prevents the audio from playing automatically

          // We need to "process" the audio to generate the media stream
          // Start the source but not connect it to speakers
          source.start();

          // Stop after the duration of the audio
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

.btn-icon {
  margin-right: 8px;
}
</style>