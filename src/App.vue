<template>
  <div id="app">
    <NavBar></NavBar>
    <DataExplorerWorkbench></DataExplorerWorkbench>
  </div>
</template>

<script>
import NavBar from './components/NavBar.vue'
import DataExplorerWorkbench from './components/DataExplorerWorkbench.vue'

export default {
  name: 'App',
  components: {
    NavBar,
    DataExplorerWorkbench
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
}
</style>
