<template>
  <FileUpload msg="Welcome to my Application!"/>
</template>

<script>
import FileUpload from './components/FileUpload.vue'

export default {
  name: 'App',
  components: {
    FileUpload
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
