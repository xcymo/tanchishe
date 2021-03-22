<template>
<div class="app">
  <HelloWorld/>
</div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'

export default {
  name: 'App',
  components: {
    HelloWorld
  }
}
</script>

<style>
*{
  margin: 0;
  padding: 0;
}
.app{
  background-color: #777;
  padding: 100px 0;
}
</style>
