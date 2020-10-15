<template>
  <div id="app">
    <router-view/>
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>

<style>
* {
  padding:0;
  margin: 0;
}
body {
  padding: 0;
  margin: 0;
}
ul,li {
  list-style-type: none;
}
a {
  text-decoration: none;
  color: black
}
input {
  outline: none;
}
</style>
