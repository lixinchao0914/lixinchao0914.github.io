<template>
  <div id="app">
    <router-view />
  </div>
</template>

<style lang="scss">
html,body{
  font-size: 14px;
  color: #333;
}
*{
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
@for $i from 1 through 5 {
  .u-line-#{$i} {
    display: -webkit-box !important;
    overflow: hidden;
    text-overflow: ellipsis;
    word-break: break-all;
    -webkit-line-clamp: $i;
    -webkit-box-orient: vertical !important;
  }
}
</style>
