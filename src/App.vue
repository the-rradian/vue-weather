<script setup>

import WeatherCard from './components/WeatherCard.vue'

</script>

<template>

  <main>
    <WeatherCard/>
    <footer>Search icon by <a target="_blank" href="https://icons8.com/icon/132/search">Icons8</a></footer>
    
  </main>
</template>

<style scoped>
body{
  display: flex;
  align-items:center;
  justify-content: center;
}
footer{
  margin-top: 100px;
  font-size: x-small;
}
</style>
