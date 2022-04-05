<script setup>
// This starter template is using Vue 3 <script setup> SFCs
// Check out https://v3.vuejs.org/api/sfc-script-setup.html#sfc-script-setup
import Crypto from "./components/Crypto.vue";
import Generator from "./components/Generator.vue";
import StreamCipher from "./components/StreamCipher.vue";
import Hello from "./components/Hello.vue";
import {computed, ref} from "vue";

const routes = {
  '/crypto': Crypto,
  '/generator': Generator,
  '/streamcipher': StreamCipher,
  '/': Hello
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
  <div id="titleContainer">
    <p id="title1">Welcome in</p><p id="title2">cryptoMaker</p>
  </div>
  <div id="navMenu">
    <button onclick="location.href='#/crypto'">Algorytmy</button>
    <button onclick="location.href='#/generator'">Generator</button>
    <button onclick="location.href='#/streamcipher'">Szyfr strumieniowy</button>
  </div>
  <component id="componentContainer" :is="currentView"/>
</template>

<style>
#app {
  background: #143547;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  position: relative;
  padding-top: 30px;
  font-family: 'Roboto', sans-serif;
  font-style: normal;
  font-weight: 200;
}

html, body {
  height: 100%;
  margin: 0;
  padding: 0;
}

button:hover {
  color: #AB2C97;
  font-size: 30px;
}

button:active {
  background: black;
  color: white;
}

button {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  padding: 10px 108px;
  height: 100%;
  border-width: 0px;
  background: #BBBBBB;
  width: 300px;
  flex: none;
  order: 0;
  flex-grow: 0;
  margin: 0px 30px;
  font-size: 26px;
}
#componentContainer {
  flex-grow: 1;
  flex-shrink: 1;
  flex-basis: auto;
  justify-content: center;
  align-items: center;
}

#titleContainer {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  flex-grow: 0;
  flex-shrink: 1;
  flex-basis: auto;
  height: 100%;
}

#title1 {
  position: static;
  left: 0%;
  right: 55.14%;
  top: 10.6%;
  bottom: 10.6%;
  font-family: 'Roboto', sans-serif;
  font-style: normal;
  font-weight: 100;
  font-size: 96px;
  line-height: 112px;
  color: #FFFFFF;
  text-shadow: 8px 8px 4px rgba(0, 0, 0, 0.25);
  flex: none;
  order: 0;
  flex-grow: 0;
  margin: 0px 10px;
}

#title2 {
  position: static;
  left: 45.72%;
  right: 0%;
  top: 0%;
  bottom: 0%;
  font-family: 'M PLUS Rounded 1c', sans-serif;
  font-style: normal;
  font-weight: 900;
  font-size: 96px;
  line-height: 143px;
  color: #FFFFFF;
  text-shadow: 8px 8px 4px rgba(243, 96, 220, 0.25);
  flex: none;
  order: 1;
  flex-grow: 0;
  margin: 0px 10px;
}

#navMenu {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  flex-grow: 0;
  flex-shrink: 1;
  flex-basis: auto;
  height: 60px;
  /*gap: 20px;*/
  background: #BBBBBB;
  width: 100%;
  margin-top: 10px;
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
