<template>
  <div>
    <div id="titleContainer">
      <p id="title1">Welcome in</p><p id="title2">cryptoMaker</p>
    </div>
    <div id="container">

      <div class="data">
        <p class="label">Podaj słowo wejściowe:</p>
        <input v-model="inputData"/>
      </div>
      <div class="errMsg"> {{errorMessageInput}}</div>

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
      <div class="errMsg"> {{errorMessageKey}}</div>

      <div class="buttonContainer">
        <button @click="encrypt">Szyfruj</button>
        <button @click="decrypt">Deszyfruj</button>
      </div>



      <div class="data">
        <p class="label">Wynik:</p>
        <p class="result">{{outputData}}</p>
      </div>
    </div>
  </div>
</template>


<script setup>
  import { ref } from 'vue'

  const inputData = ref('')
  const key = ref('')
  const outputData = ref('')

  const selected = ref('Rail Fence')
  const options = ref([
      "Rail Fence",
      "Przestawienia macierzowe - klucz liczbowy",
      "Przestawienia macierzowe - klucz słowny v1",
      "Przestawienia macierzowe - klucz słowny v2",
      "Szyfr Cezara",
      "Szyfrowanie Vigenere’a"
  ])

  const errorMessageInput = ref('')
  const errorMessageKey = ref('')

  function testInputData() {
    errorMessageInput.value = ''
    if (inputData.value.length === 0) {
      errorMessageInput.value = "Pole ze słowem wejściowym nie może być puste";
      return false;
    } else {
      const reg = /^[a-zA-Z]+$/;
      if (!reg.test(inputData.value)) {
        errorMessageInput.value = "Użyto niedozwolonych znaków - używaj tylko liter";
        return false;
      }
    }
    return true;
  }

  function testKey() {
    errorMessageKey.value = ''
    if (key.value.length === 0) {
      errorMessageKey.value = "Pole z kluczem nie może być puste";
      return false;
    } else switch (selected.value) {
      case 'Rail Fence':
        break;
      case 'Przestawienia macierzowe - klucz liczbowy':
        break;
      case 'Przestawienia macierzowe - klucz słowny v1':
        break;
      case 'Przestawienia macierzowe - klucz słowny v2':
        break;
      case 'Szyfr Cezara':
        const reg = /^[0-9]+$/;
        if (!reg.test(key.value)) {
          errorMessageKey.value = "Użyto niedozwolonych znaków - podaj tylko liczbę";
          return false;
        }
        break;
      case 'Szyfrowanie Vigenere’a':
        break;
      default:
        outputData.value = 'Nie udało się zaszyfrować'
        break;
    }
    return true;
  }

  function encrypt() {
    outputData.value = ''
    if (testInputData() && testKey()) {
      switch (selected.value) {
        case 'Rail Fence':
          encryptRailFence();
          break;
        case 'Przestawienia macierzowe - klucz liczbowy':
          encryptMatrixWithNumbers();
          break;
        case 'Przestawienia macierzowe - klucz słowny v1':
          encryptMatrixWithWordV1();
          break;
        case 'Przestawienia macierzowe - klucz słowny v2':
          encryptMatrixWithWordV2();
          break;
        case 'Szyfr Cezara':
          encryptCaesarCipher();
          break;
        case 'Szyfrowanie Vigenere’a':
          encryptVigenereEncryption();
          break;
        default:
          outputData.value = 'Nie udało się zaszyfrować'
          break;
      }
    }
  }

  function encryptRailFence() {
    outputData.value = 'zaszyfrowano railfence'
  }

  function encryptMatrixWithNumbers() {
    outputData.value = 'przestawienia macierzowe - klucz liczbowy'
  }

  function encryptMatrixWithWordV1() {
    outputData.value = 'zaszyfrowano przestawienia macierzowe - klucz słowny v1'
  }

  function encryptMatrixWithWordV2() {
    outputData.value = 'zaszyfrowano przestawienia macierzowe - klucz słowny v2'
  }

  function encryptCaesarCipher() {
    let n = 26
    let k = parseInt(key.value)
    let result = ''
    let alphabet = ["A", "B", "C", "D", "E",
                    "F", "G", "H", "I", "J",
                    "K", "L", "M", "N", "O",
                    "P", "Q", "R", "S", "T",
                    "U", "V", "W", "X", "Y", "Z"]
    for (let i = 0; i < inputData.value.length; i++) {
      result += alphabet[(alphabet.indexOf(inputData.value[i].toUpperCase()) + 1 + k) % n - 1]
    }
    outputData.value = result
  }

  function encryptVigenereEncryption() {
    outputData.value = 'zaszyfrowano szyfrowanie Vigenere’a'
  }

  function decrypt() {
    outputData.value = ''
    if (testInputData() && testKey()) {
      switch (selected.value) {
        case 'Rail Fence':
          decryptRailFence();
          break;
        case 'Przestawienia macierzowe - klucz liczbowy':
          decryptMatrixWithNumbers();
          break;
        case 'Przestawienia macierzowe - klucz słowny v1':
          decryptMatrixWithWordV1();
          break;
        case 'Przestawienia macierzowe - klucz słowny v2':
          decryptMatrixWithWordV2();
          break;
        case 'Szyfr Cezara':
          decryptCaesarCipher();
          break;
        case 'Szyfrowanie Vigenere’a':
          decryptVigenereEncryption();
          break;
        default:
          outputData.value = 'Nie udało się odszyfrować'
          break;
      }
    }
  }

  function decryptRailFence() {
    outputData.value = 'odszyfrowano railfence'
  }

  function decryptMatrixWithNumbers() {
    outputData.value = 'przestawienia macierzowe - klucz liczbowy'
  }

  function decryptMatrixWithWordV1() {
    outputData.value = 'odszyfrowano przestawienia macierzowe - klucz słowny v1'
  }

  function decryptMatrixWithWordV2() {
    outputData.value = 'odszyfrowano przestawienia macierzowe - klucz słowny v2'
  }

  function decryptCaesarCipher() {
    let n = 26
    let k = parseInt(key.value)
    let result = ''
    let alphabet = ["A", "B", "C", "D", "E",
      "F", "G", "H", "I", "J",
      "K", "L", "M", "N", "O",
      "P", "Q", "R", "S", "T",
      "U", "V", "W", "X", "Y", "Z"]
    for (let i = 0; i < inputData.value.length; i++) {
      result += alphabet[(alphabet.indexOf(inputData.value[i].toUpperCase()) + 1 + n - k) % n - 1]
    }
    outputData.value = result
  }

  function decryptVigenereEncryption() {
    outputData.value = 'odszyfrowano szyfrowanie Vigenere’a'
  }

</script>

<style scoped>

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

    font-family: 'Roboto', sans-serif;
    font-style: normal;
    font-weight: 200;
    font-size: 36px;
  }

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
    background: #BBBBBB;
    box-shadow: 4px 4px 8px 4px rgba(0, 0, 0, 0.25);
    flex: none;
    order: 0;
    flex-grow: 0;
    margin: 0px 30px;
    font-size: 36px;
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
    box-shadow: 4px 4px 16px 4px rgba(0, 0, 0, 0.25);
    flex: none;
    order: 1;
    flex-grow: 0;
    margin: 0px 4px;
    font-size: 30px;
  }

  .errMsg {
    font-size: 16px;
    color: red;
  }

  .result {
    padding-top: 15px;
    padding-left: 5px;
  }

  .label {
    position: static;
    left: 0.28%;
    right: 63.93%;
    top: 11.11%;
    bottom: 11.11%;
    width: 383px;
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
