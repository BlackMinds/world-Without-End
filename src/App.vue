<template>
  <div id="app">
    <router-view />
  </div>
</template>

<script>
export default {
  name: "App",
};
</script>

<style>
* {
  padding: 0;
  margin: 0;
}
body {
  padding: 0;
  margin: 0;
}
ul,
li {
  list-style-type: none;
}
a {
  text-decoration: none;
  color: black;
}
input {
  outline: none;
}
.ivu-poptip-popper .ivu-poptip-content .ivu-poptip-inner {
  /* background: rgb(18 19 22) !important; */
  background: linear-gradient(to bottom, #000000 0%, #9d1334 100%) !important;
}
</style>
