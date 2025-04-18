<template>
  <div class="theme-toggle">
    <button @click="toggleTheme" class="theme-btn" :class="{ 'light-btn': theme === 'light', 'dark-btn': theme === 'dark' }">
      <span v-if="theme === 'dark'" class="theme-icon">☀️</span>
      <span v-else class="theme-icon">🌙</span>
      {{ theme === 'dark' ? 'Light Mode' : 'Dark Mode' }}
    </button>
  </div>
</template>

<script>
export default {
  name: 'ThemeToggle',
  props: {
    theme: {
      type: String,
      required: true
    },
    toggleTheme: {
      type: Function,
      required: true
    }
  }
};
</script>

<style scoped>
.theme-toggle {
  margin-right: 10px;
}

.theme-btn {
  padding: 8px 16px;
  border: none;
  border-radius: 20px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
}

.dark-btn {
  background: linear-gradient(45deg, #333, #555);
  color: white;
}

.light-btn {
  background: linear-gradient(45deg, #e0e0e0, #ffffff);
  color: #333;
}

.theme-icon {
  margin-right: 8px;
  font-size: 1.1rem;
}
</style>
