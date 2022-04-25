<template>
  <div>
    <div id="container1">
      <div id="container2">
        <div class="data">
          <p class="label">Wybierz plik:</p>
          <input class="fileInput" type="file" @change="onFileSelected"/>
        </div>
        <p></p>
        <div class="errMsg"> {{ errorMessageNoFileInput }}</div>
          <div class="buttonContainer">
            <button @click="startDES">Start DES</button>
          </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import {ref} from 'vue'

const errorMessageInput = ref('')
const errorMessageNoFileInput = ref('')
const selectedFile = ref(null)
const blocksArray = ref([])

let dataBlock = ref([
    1, 0, 1, 1, 1, 0, 1, 0,
    1, 0, 1, 1, 1, 0, 0, 0,
    0, 1, 0, 1, 1, 0, 1, 1,
    0, 0, 1, 1, 1, 0, 1, 1,
    1, 0, 1, 1, 0, 0, 1, 0,
    0, 0, 0, 1, 0, 0, 1, 0,
    1, 1, 1, 1, 1, 0, 0, 1,
    1, 1, 1, 1, 1, 0, 1, 0])

const initialPermutation = ref([
  58, 50, 42, 34, 26, 18, 10, 2,
  60, 52, 44, 36, 28, 20, 12, 4,
  62, 54, 46, 38, 30, 22, 14, 6,
  64, 56, 48, 40, 32, 24, 16, 8,
  57, 49, 41, 33, 25, 17, 9, 1,
  59, 51, 43, 35, 27, 19, 11, 3,
  61, 53, 45, 37, 29, 21, 13, 5,
  63, 55, 47, 39, 31, 23, 15, 7])

function showTable(arr) {
  let table = '';
  for (let i = 0; i < arr.length; i++) {
    table += arr[i] + " "
  }
  console.log(table);
}

function startDES(event){
  makeInitialPermutation()
}

function makeInitialPermutation() {
  // tworzymy tablicę pomocniczą przechowującą aktualne wartości 64-bitowego bloku:
  let tmp = [];
  for (let i = 0; i < dataBlock.value.length; i++) {
    tmp[i] = dataBlock.value[i]
  }

  console.log("Blok początkowy:")
  showTable(dataBlock.value);

  // permutacja 'initial permutation' 64-bitowego bloku wejściowego:
  for (let i = 0; i < dataBlock.value.length; i++) {
    dataBlock.value[i] = tmp[initialPermutation.value[i]-1]
  }

  console.log("Blok po permutacji 'initial permutation': ")
  showTable(dataBlock.value);
}


function onFileSelected(event) {
  selectedFile.value = event.target.files[0]
  createBlocks()
  console.log(selectedFile.value)

}

const createBlocks = () => {
  blocksArray.value = []
  const fileReader = new FileReader()
  fileReader.readAsArrayBuffer(selectedFile.value)

  fileReader.onload = () => {
    let buffer = new Uint8Array(fileReader.result)
    console.log("przed jakimikolwiek modyfikacjami - plus długość tablicy" + buffer.length)
    console.log(buffer)
    buffer = Array.from(buffer)

    if (buffer.length % 8 !== 0) buffer = completeTheBlock(buffer)
    else buffer.push(1)

    blocksArray.value = toBinaryArray(buffer)
    console.log("finalny wynik - plus długość tablicy" + blocksArray.value.length)
    console.log(blocksArray.value)
  }

}

function completeTheBlock(buffer) {
  let i = 0
  while (buffer.length % 8 !== 0) {
    buffer.push(1)
    ++i
  }

  buffer[buffer.length - 1] = i

  console.log("po dodaniu brakujących bajtów - plus długość tablicy" + buffer.length)
  console.log(buffer)

  return buffer
}

function toBinaryArray(buffer) {
  let arrayOfBlocks = []
  let i = 0;
  let array = []

  buffer.forEach((v) => {
    if (i % 8 === 0 && !(buffer.length % 8 !== 0 && i === buffer.length - 1)) array = []
    let string = v.toString(2)

    while (string.length !== 8) {
      string = "0" + string
    }

    for (let i = 0; i < string.length; i++) {
      array.push(parseInt(string.charAt(i)))
    }

    ++i
    if (array.length >= 64 && !(buffer.length % 8 !== 0 && i === buffer.length - 2)) arrayOfBlocks.push(array)
  })


  return arrayOfBlocks
}

</script>

<style scoped>

#container1 {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  position: static;
  min-width: 1168px;
  width: auto;
  height: 520px;
  /*left: 352px;*/
  /*top: 388px;*/
  background: #E5E5E5;
  box-shadow: 4px 4px 16px 8px rgba(0, 0, 0, 0.25);
  /*flex: none;*/
  /*align-self: stretch;*/
  flex-grow: 1;
  flex-shrink: 0;
  flex-basis: auto;
  margin: 30px 0px;

  font-family: 'Roboto', sans-serif;
  font-style: normal;
  font-weight: 200;
  font-size: 36px;
}
#container3 {
  /*display: flex;*/
  /*background: #42b983;*/
  justify-content: center;
  align-items: center;
  position: relative;
  flex-grow: 0;
  flex-shrink: 1;
  flex-basis: auto;
  margin: 30px 0;
  height: 100%;
}

#container2 {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  position: relative;
  height: 100%;
  flex-grow: 1;
  flex-shrink: 1;
  flex-basis: auto;
  font-family: 'Roboto', sans-serif;
  font-style: normal;
  font-weight: 200;
  font-size: 36px;
}

button:hover {
  background: #AB2C97;
  color: black;
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
  width: auto;
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
  width: auto;
  height: 54px;
  top: 112px;
  flex: none;
  flex-grow: 0;
  margin: 18px 0px;
}

input, .result, select {
  position: static;
  width: auto;
  height: 54px;
  border-width: 0px;
  background: #FFFFFF;
  box-shadow: 4px 4px 16px 4px rgba(0, 0, 0, 0.25);
  /*flex: none;*/
  /*order: 1;*/
  flex-grow: 0;
  /*margin: 0px 4px;*/
  font-size: 30px;
}

.steps {
  height: 90%;
  width: auto;
  text-align: center;
  /*background: #E5E5E5;*/
  border-width: 0;
  margin: 30px;
  word-wrap: break-word;
  /*padding: 30px;*/
  /*justify-self: center;*/
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
  /*left: 0.28%;*/
  /*right: 63.93%;*/
  /*top: 11.11%;*/
  /*bottom: 11.11%;*/
  width: 383px;
  line-height: 42px;
  color: #000000;
  text-shadow: 8px 8px 4px rgba(0, 0, 0, 0.25);
  flex: none;
  order: 0;
  flex-grow: 0;
  margin: 0px 4px;
}

.fileInput {
  background: #E5E5E5;
  box-shadow: 4px 4px 16px 4px rgba(0, 0, 0, 0);
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

</style>
