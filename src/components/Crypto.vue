<template>
  <div>
    <div id="titleContainer">
      <p id="title1">Welcome in</p><p id="title2">cryptoMaker</p>
    </div>
    <div id="container">

      <div class="data">
        <p class="label">Podaj słowo wejściowe:</p>
        <input v-model="inputData" />
      </div>

      <div class="data">
        <p class="label">Wybierz algorytm</p>
        <select v-model="selected">
          <option v-for="option in options" :value="option">
            {{ option }}
          </option>
        </select>

      </div>
      <div class="data">
        <p class="label">Podaj klucz:</p>
        <input v-model="key"/>
      </div>
      <div class="buttonContainer">
        <button>Szyfruj</button>
        <button>Deszyfruj</button>
      </div>
      <div class="data">
        <p class="label">Wynik:</p>
        <p class="result"/>
      </div>
    </div>
  </div>
</template>


<script setup>
  import { ref } from 'vue'

  const inputData = ref('')
  const key = ref('')

  const selected = ref('Rail Fence')
  const options = ref([
      "Rail Fence",
      "B",
      "C"
  ])
</script>

<style scoped>
  button:hover {
    background: #AB2C97;
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
    padding: 6px 108px;
    border-width: 0px;
    position: static;
    width: 231px;
    height: 54px;
    font-size: 36px;
    background: #BBBBBB;
    box-shadow: 4px 4px 8px 4px rgba(0, 0, 0, 0.25);
    flex: none;
    order: 0;
    flex-grow: 0;
    margin: 0px 30px;
  }

  .buttonContainer {
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;
    padding: 0px;

    position: static;
    width: 492px;
    height: 54px;
    left: 338px;
    top: 112px;
    flex: none;
    flex-grow: 0;
    margin: 18px 0px;
  }

  input, .result, select {
    position: static;
    left: 37.01%;
    right: 0.84%;
    top: 0%;
    bottom: 0%;
    width: 665px;
    height: 54px;
    border-width: 0px;
    background: #FFFFFF;
    font-size: 36px;
    box-shadow: 4px 4px 16px 4px rgba(0, 0, 0, 0.25);
    flex: none;
    order: 1;
    flex-grow: 0;
    margin: 0px 4px;
  }

  .label {
    position: static;
    left: 0.28%;
    right: 63.93%;
    top: 11.11%;
    bottom: 11.11%;
    width: 383px;
    font-family: 'Roboto', sans-serif;
    font-style: normal;
    font-weight: 200;
    font-size: 36px;
    line-height: 42px;
    color: #000000;
    text-shadow: 8px 8px 4px rgba(0, 0, 0, 0.25);
    flex: none;
    order: 0;
    flex-grow: 0;
    margin: 0px 4px;
  }

  .data {
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;
    padding: 0px 5px;
    position: static;
    width: 1070px;
    height: 54px;
    left: 49px;
    top: 40px;
    flex: none;
    order: 0;
    flex-grow: 0;
    margin: 18px 0px;
  }

  #container {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    position: static;
    width: 1168px;
    height: 520px;
    left: 352px;
    top: 388px;
    background: #E5E5E5;
    box-shadow: 4px 4px 16px 8px rgba(0, 0, 0, 0.25);
    flex: none;
    order: 1;
    align-self: stretch;
    flex-grow: 0;
    margin: 30px 0px;
  }

  #titleContainer {
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;
    padding: 0px;
    position: static;
    width: 1168px;
    height: 151px;
    left: 368px;
    top: 75px;
    flex: none;
    order: 0;
    flex-grow: 0;
    margin: 20px 0px;
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
</style>
