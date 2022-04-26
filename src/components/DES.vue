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
const blocksArray = ref([]) //tu jest tablica gdzie będą wszystkie bloki

let key = [
  0, 1, 0, 1, 1, 0, 1, 0,
  1, 0, 1, 1, 1, 0, 0, 0,
  0, 1, 0, 1, 1, 1, 1, 1,
  0, 0, 0, 1, 1, 0, 1, 0,
  1, 0, 1, 1, 0, 0, 1, 0,
  0, 0, 0, 1, 0, 0, 1, 0,
  1, 0, 1, 1, 1, 0, 0, 0,
  0, 0, 1, 1, 1, 0, 1, 0
]
let dataBlock = [
    1, 0, 1, 1, 1, 0, 1, 0,
    1, 0, 1, 1, 1, 0, 0, 0,
    0, 1, 0, 1, 1, 0, 1, 1,
    0, 0, 1, 1, 1, 0, 1, 1,
    1, 0, 1, 1, 0, 0, 1, 0,
    0, 0, 0, 1, 0, 0, 1, 0,
    1, 1, 1, 1, 1, 0, 0, 1,
    1, 1, 1, 1, 1, 0, 1, 0
]
const initialPermutation = [
  58, 50, 42, 34, 26, 18, 10, 2,
  60, 52, 44, 36, 28, 20, 12, 4,
  62, 54, 46, 38, 30, 22, 14, 6,
  64, 56, 48, 40, 32, 24, 16, 8,
  57, 49, 41, 33, 25, 17, 9, 1,
  59, 51, 43, 35, 27, 19, 11, 3,
  61, 53, 45, 37, 29, 21, 13, 5,
  63, 55, 47, 39, 31, 23, 15, 7
]
const permutedChoice = [
  57, 49, 41, 33, 25, 17, 9,
  1, 58, 50, 42, 34, 26, 18,
  10, 2, 59, 51, 43, 35, 27,
  19, 11, 3, 60, 52, 44, 36,
  63, 55, 47, 39, 31, 23, 15,
  7, 62, 54, 46, 38, 30, 22,
  14, 6, 61, 53, 45, 37, 29,
  21, 13, 5, 28, 20, 12, 4
]
const permutedChoice2 = [
  14, 17, 11, 24, 1, 5,
  3, 28, 15, 6, 21, 10,
  23, 19, 12, 4, 26, 8,
  16, 7, 27, 20, 13, 2,
  41, 52, 31, 37, 47, 55,
  30, 40, 51, 45, 33, 48,
  44, 49, 39, 56, 34, 53,
  46, 42, 50, 36, 29, 32
]
const shiftTableForEachIteration = [
  1, 1, 2, 2,
  2, 2, 2, 2,
  1, 2, 2, 2,
  2, 2, 2, 1
]
const extensionArray = [
  32, 1, 2, 3, 4, 5,
  4, 5, 6, 7, 8, 9,
  8, 9, 10, 11, 12, 13,
  12, 13, 14, 15, 16, 17,
  16, 17, 18, 19, 20, 21,
  20, 21, 22, 23, 24, 25,
  24, 25, 26, 27, 28, 29,
  28, 29, 30, 31, 32, 1
]
const SnArray = [
    // S1:
    [14, 4, 13, 1, 2, 15, 11, 8, 3, 10, 6, 12, 5, 9, 0, 7,
      0, 15, 7, 4, 14, 2, 13, 1, 10, 6, 12, 11, 9, 5, 3, 8,
      4, 1, 14, 8, 13, 6, 2, 11, 15, 12, 9, 7, 3, 10, 5, 0,
      15, 12, 8, 2, 4, 9, 1, 7, 5, 11, 3, 14, 10, 0, 6, 13],
    // S2:
    [15, 1, 8, 14, 6, 11, 3, 4, 9, 7, 2, 13, 12, 0, 5, 10,
      3, 13, 4, 7, 15, 2, 8, 14, 12, 0, 1, 10, 6, 9, 11, 5,
      0, 14, 7, 11, 10, 4, 13, 1, 5, 8, 12, 6, 9, 3, 2, 15,
      13, 8, 10, 1, 3, 15, 4, 2, 11, 6, 7, 12, 0, 5, 14, 9
    ],
    // S3:
    [
      10, 0, 9, 14, 6, 3, 15, 5, 1, 13, 12, 7, 11, 4, 2, 8,
      13, 7, 0, 9, 3, 4, 6, 10, 2, 8, 5, 14, 12, 11, 15, 1,
      13, 6, 4, 9, 8, 15, 3, 0, 11, 1, 2, 12, 5, 10, 14, 7,
      1, 10, 13, 0, 6, 9, 8, 7, 4, 15, 14, 3, 11, 5, 2, 12
    ],
    // S4:
    [
      7, 13, 14, 3, 0, 6, 9, 10, 1, 2, 8, 5, 11, 12, 4, 15,
      13, 8, 11, 5, 6, 15, 0, 3, 4, 7, 2, 12, 1, 10, 14, 9,
      10, 6, 9, 0, 12, 11, 7, 13, 15, 1, 3, 14, 5, 2, 8, 4,
      3, 15, 0, 6, 10, 1, 13, 8, 9, 4, 5, 11, 12, 7, 2, 14
    ],
    // S5:
    [
      2, 12, 4, 1, 7, 10, 11, 6, 8, 5, 3, 15, 13, 0, 14, 9,
      14, 11, 2, 12, 4, 7, 13, 1, 5, 0, 15, 10, 3, 9, 8, 6,
      4, 2, 1, 11, 10, 13, 7, 8, 15, 9, 12, 5, 6, 3, 0, 14,
      11, 8, 12, 7, 1, 14, 2, 13, 6, 15, 0, 9, 10, 4, 5, 3
    ],
    // S6:
    [
      12, 1, 10, 15, 9, 2, 6, 8, 0, 13, 3, 4, 14, 7, 5, 11,
      10, 15, 4, 2, 7, 12, 9, 5, 6, 1, 13, 14, 0, 11, 3, 8,
      9, 14, 15, 5, 2, 8, 12, 3, 7, 0, 4, 10, 1, 13, 11, 6,
      4, 3, 2, 12, 9, 5, 15, 10, 11, 14, 1, 7, 6, 0, 8, 13
    ],
    // S7:
    [
      4, 11, 2, 14, 15, 0, 8, 13, 3, 12, 9, 7, 5, 10, 6, 1,
      13, 0, 11, 7, 4, 9, 1, 10, 14, 3, 5, 12, 2, 15, 8, 6,
      1, 4, 11, 13, 12, 3, 7, 14, 10, 15, 6, 8, 0, 5, 9, 2,
      6, 11, 13, 8, 1, 4, 10, 7, 9, 5, 0, 15, 14, 2, 3, 12
    ],
    // S8:
    [
      13, 2, 8, 4, 6, 15, 11, 1, 10, 9, 3, 14, 5, 0, 12, 7,
      1, 15, 13, 8, 10, 3, 7, 4, 12, 5, 6, 11, 0, 14, 9, 2,
      7, 11, 4, 1, 9, 12, 14, 2, 0, 6, 10, 13, 15, 3, 5, 8,
      2, 1, 14, 7, 4, 10, 8, 13, 15, 12, 9, 0, 3, 5, 6, 11
    ]
]

function showBlock(block) {
  let bits = '';
  for (let i = 0; i < block.length; i++) {
    bits += block[i] + " "
  }
  console.log(bits);
}

function startDES(event){
  let leftDataBlock, rightDataBlock, CKeyBlock, DKeyBlock, xorBlock

  // tu pewnie będzie: for(block in blocks):

  console.log("Blok początkowy:")
  showBlock(dataBlock)

  dataBlock = makeInitialPermutation(dataBlock)
  console.log("Blok po permutacji 'initial permutation': ")
  showBlock(dataBlock)

  console.log("Klucz początkowy: ")
  // zamieniam klucz na zwykłe liczby od 1 do 64 tylko po to żeby łatwiej sprawdzać poprawność permutacji
  // for(let i = 0; i < 64; i++) {
  //   key[i] = i+1
  // }
  showBlock(key)

  key = makePermutedChoice(key)
  console.log("Przetworzony klucz 56-bitowy permutacją 'permuted choice': ")
  showBlock(key)

  for (let i = 0; i < 1; i++) {
    leftDataBlock = dataBlock.slice(0, dataBlock.length/2)
    rightDataBlock = dataBlock.slice(dataBlock.length/2)
    console.log("Lewy blok: ")
    showBlock(leftDataBlock)
    console.log("Prawy blok: ")
    showBlock(rightDataBlock)

    CKeyBlock = key.slice(0, key.length/2)
    DKeyBlock = key.slice(key.length/2)
    console.log("Pierwsza część klucza C: ")
    showBlock(CKeyBlock)
    console.log("Druga część klucza D: ")
    showBlock(DKeyBlock)

    CKeyBlock = makeLeftShift(CKeyBlock, shiftTableForEachIteration[i])
    console.log("Pierwsza część klucza C po 1. przesunięciu w lewo: ")
    showBlock(CKeyBlock)
    DKeyBlock = makeLeftShift(DKeyBlock, shiftTableForEachIteration[i])
    console.log("Pierwsza część klucza D po 1. przesunięciu w lewo: ")
    showBlock(DKeyBlock)

    key = CKeyBlock.concat(DKeyBlock)
    console.log("Połączone części klucza C i D: ")
    showBlock(key)

    key = makePermutedChoice2(key)
    console.log("Przetworzony klucz 48-bitowy permutacją 'permuted choice 2': ")
    showBlock(key)

    rightDataBlock = makeExtensionArrayPermutation(rightDataBlock)
    console.log("Przetworzona prawa część bloku wejściowego za pomocą 'extension array': ")
    showBlock(rightDataBlock)

    xorBlock = makeXORforBlocks(key, rightDataBlock)
    console.log("48-bitowy ciąg uzyskany za pomocą operacji XOR na 48-bitowym kluczu i 48-bitowej wersji prawego bloku: ")
    showBlock(xorBlock)

    xorBlock = makeBlockTransformationsBySnArray(xorBlock)
    console.log("32-bitowy ciąg uzyskany za pomocą przekształceń za pomocą tablic S 48-bitowego ciągu uzyskanego po operacji XOR: ")
    showBlock(xorBlock)
  }
}

function makeInitialPermutation(block) {
  // stworzenie tablicy pomocniczej przechowującej aktualne wartości 64-bitowego bloku:
  let tmp = [];
  for (let i = 0; i < block.length; i++) {
    tmp[i] = block[i]
  }

  // permutacja 'initial permutation' 64-bitowego bloku wejściowego:
  for (let i = 0; i < block.length; i++) {
    block[i] = tmp[initialPermutation[i]-1]
  }

  return block
}

function makePermutedChoice(block) {
  // stworzenie nowej tablicy do przechowywania wartości ze zredukowanej tablicy wejściowej
  // za pomocą permutacji 'permuted choice':
  let reducedBlock = []
  for (let i = 0; i < permutedChoice.length; i++) {
    reducedBlock[i] = block[permutedChoice[i]-1]
  }
  return reducedBlock
}

function makePermutedChoice2(block) {
  // stworzenie nowej tablicy do przechowywania wartości ze zredukowanej tablicy wejściowej
  // za pomocą permutacji 'permuted choice 2':
  let reducedBlock = []
  for (let i = 0; i < permutedChoice2.length; i++) {
    reducedBlock[i] = block[permutedChoice2[i]-1]
  }
  return reducedBlock
}

function makeLeftShift(block, numberOfBits) {
  // zapamiętanie pierwszych n bitów bloku wejściowego, gdzie n to liczba bitów o które przesuwamy:
  let tmp = []
  for (let i = 0; i < numberOfBits; i++) {
    tmp[i] = block[i]
  }

  // przesunięcie wszystkich bitów o n bitów w lewo, poza n ostatnimi, gdzie n to liczba bitów o które przesuwamy:
  for (let i = 0; i < block.length - numberOfBits; i++) {
    block[i] = block[i + numberOfBits]
  }

  // przypisanie do bloku wyjściowego na końcu n zapamiętanych w kroku wyżej pierwszych bitów bloku wejściowego, gdzie n
  // to liczba bitów o które przesuwamy:
  for (let i = 0; i < numberOfBits; i++) {
    block[block.length - numberOfBits + i] = tmp[i]
  }

  return block
}

function makeExtensionArrayPermutation(block) {
  let extensionBlock = []
  for (let i = 0; i < extensionArray.length; i++) {
    extensionBlock[i] = block[extensionArray[i]-1]
  }
  return extensionBlock
}

function XOR(p, q) {
  if (p === q) return 0;
  else return 1;
}

function makeXORforBlocks(firstBlock, secondBlock) {
  let xorBlock = []
  for (let i = 0; i < firstBlock.length; i++) {
    xorBlock[i] = XOR(firstBlock[i], secondBlock[i])
  }
  return xorBlock
}

function makeBlockTransformationsBySnArray(block) {
  let partsOfBlock = []
  for (let i = 0; i < 8; i++ ) {
    partsOfBlock[i] = block.slice(6 * i, (6 * i) + 6)
  }

  // Sprawdzam poprawność "pocięcia bloku"
  console.log("Block: ")
  showBlock(block)
  console.log("Pociety blok:")
  for (let i = 0; i < 8; i++ ) {
    showBlock(partsOfBlock[i])
  }

  let rowNr, colNr
  for (let i = 0; i < 1; i++) {
    rowNr = parseInt(partsOfBlock[i][0] + "" + partsOfBlock[i][5], 2)
    console.log("row nr in binary: " + partsOfBlock[i][0] + partsOfBlock[i][5] + " and in decimal " + rowNr)
  }

  return block
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

  fileReader.onload = () => { //wczytujemy plik
    let buffer = new Uint8Array(fileReader.result) //pobieramy tablice bajtów
    console.log("przed jakimikolwiek modyfikacjami - plus długość tablicy" + buffer.length)
    console.log(buffer)
    buffer = Array.from(buffer)

    if (buffer.length % 8 !== 0) buffer = completeTheBlock(buffer) //gdy nie można podzielić równo po 64 bity
    else buffer.push(1) // w przeciwnym wypadku dodajemy jeden bajt więcej by program wiedział że na starcie była równa ilość

    blocksArray.value = toBinaryArray(buffer) // tutaj ustawiamy wartość dla tablicy bloków
    console.log("finalny wynik - plus długość tablicy" + blocksArray.value.length)
    console.log(blocksArray.value)
  }

}

function completeTheBlock(buffer) {
  let i = 0
  while (buffer.length % 8 !== 0) { //uzupełniamy blok w przypadku gdy ilość bajtów nie dzieli się przez 8
    buffer.push(1)
    ++i
  }

  buffer[buffer.length - 1] = i //ostatni bajt zawiera info ile zostało dodanych bajtów

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
    let string = v.toString(2)  //zamieniamy aktualny bajt na postać dwójkową

    while (string.length !== 8) {
      string = "0" + string //dodajemy zera tak by string posiadał 8 znaków
    }

    for (let i = 0; i < string.length; i++) {
      array.push(parseInt(string.charAt(i))) //zamieniamy na inty i wrzucamy do tablicy
    }

    //zostanie to umieszczone w tablicy bloków wtedy, gdy nasz array będzie mieć długość 64 lub więcej
    if (array.length >= 64 && !(buffer.length % 8 !== 0 && i === buffer.length - 2)) arrayOfBlocks.push(array)
    ++i
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
