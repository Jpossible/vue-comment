<template>
  <div class="comment-container">
    <j-comment />
  </div>
</template>

<script>
import JComment from '@/components/JComment'
export default {
  name: 'app',
  components: {
    JComment
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
