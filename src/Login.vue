<template>
    <div>
      <NewsList msg="Welcome to Your Vue.js + TypeScript App" />
    </div>
  </template>
  
  <script lang="ts">
  import { Component, Vue } from "vue-property-decorator";
  import NewsList from "./components/NewsList.vue";
  
  @Component({
    components: {
      NewsList
    }
  })
  export default class App extends Vue {}
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
  