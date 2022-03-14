<script setup>
// This starter template is using Vue 3 <script setup> SFCs
// Check out https://v3.vuejs.org/api/sfc-script-setup.html#sfc-script-setup
import HelloWorld from './components/HelloWorld.vue'
import Crypto from "./components/Crypto.vue";
</script>

<template>
  <Crypto/>
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
