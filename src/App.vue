<template>
  <div class="mt-0" id="app">
    <PageHeader></PageHeader>
    <HomePage/>
  </div>
</template>

<script>
import HomePage from './components/HomePage.vue'
import PageHeader from './components/PageHeader.vue'

export default {
  name: 'App',
  components: {
    HomePage,
    PageHeader
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
