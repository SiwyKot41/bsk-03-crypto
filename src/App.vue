<script setup>
// This starter template is using Vue 3 <script setup> SFCs
// Check out https://v3.vuejs.org/api/sfc-script-setup.html#sfc-script-setup
import HelloWorld from './components/HelloWorld.vue'
import Crypto from "./components/Crypto.vue";
import Generator from "./components/Generator.vue";
import {computed, ref} from "vue";

const routes = {
  '/': Crypto,
  '/generator': Generator
}

const currentPath = ref(window.location.hash)

window.addEventListener('hashchange', () => {
  currentPath.value = window.location.hash
})

const currentView = computed(() => {
  return routes[currentPath.value.slice(1) || '/']
})

</script>

<template>
  <a href="#/">Crypto</a> |
  <a href="#/generator">Generator</a> |
  <component :is="currentView"/>
</template>

<style>
#app {
  background: #143547;
  width: 100%;
  min-height: 100%;

  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  /*padding: 75px 352px;*/

  position: relative;
}

html, body {
  height: 100%;
  margin: 0;
  padding: 0;
}

@font-face {
  font-family: "Roboto";
  src: local("Roboto"),
  url(./fonts/Roboto/Roboto-Thin.ttf) format("truetype");

  font-family: "Rounded Mplus 1c";
  src: local("Rounded"),
  url(./fonts/M_PLUS_Rounded_1c/MPLUSRounded1c-Regular.ttf) format("truetype");
}
</style>
