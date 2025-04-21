<template>
  <button class="custom-btn" @click="$emit('apply-grayscale')">
    ⚫ Grayscale
  </button>
</template>

<script>
export default {
  name: 'GrayscaleButton'
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

