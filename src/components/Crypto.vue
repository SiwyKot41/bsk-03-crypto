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
import {ref} from 'vue'

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

  const alphabet = ["A", "B", "C", "D", "E",
    "F", "G", "H", "I", "J",
    "K", "L", "M", "N", "O",
    "P", "Q", "R", "S", "T",
    "U", "V", "W", "X", "Y", "Z"]

  const errorMessageInput = ref('')
  const errorMessageKey = ref('')

  function testInputData() {
    errorMessageInput.value = ''
    const reg = /^[ ]+$/

    if (inputData.value.length === 0 || reg.test(inputData.value)) {
      errorMessageInput.value = "Pole ze słowem wejściowym nie może być puste";
      return false;
    }

    const reg2 = /^[a-zA-Z ]+$/;
    if (!reg2.test(inputData.value)) {
      errorMessageInput.value = "Użyto niedozwolonych znaków - używaj tylko liter";
      return false;
    }

    return true;
  }

  function testKey() {
    errorMessageKey.value = ''
    const reg = /^[ ]+$/
    if (key.value.length === 0 || reg.test(key.value)) {
      errorMessageKey.value = "Pole z kluczem nie może być puste";
      return false;
    } else switch (selected.value) {
      case 'Rail Fence':
        const reg3 = /^[0-9]+$/;
        if (!reg3.test(key.value)) {
          errorMessageKey.value = "Użyto niedozwolonych znaków - podaj tylko liczbę";
          return false;
        }
        break;
      case 'Przestawienia macierzowe - klucz liczbowy':
        break;
      case 'Przestawienia macierzowe - klucz słowny v1':
        const reg2 = /^[a-zA-Z]+$/;
        if (!reg2.test(key.value)) {
          errorMessageKey.value = "Użyto niedozwolonych znaków - używaj tylko liter";
          return false;
        }
          break
      case 'Przestawienia macierzowe - klucz słowny v2':
        break;
      case 'Szyfr Cezara':
        const reg4 = /^[0-9]+$/;
        if (!reg4.test(key.value)) {
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
    outputData.value = 'railfence odszyfrowywanie'
  }

  function encryptMatrixWithNumbers() {
    outputData.value = 'przestawienia macierzowe - klucz liczbowy'
  }

  function encryptMatrixWithWordV1() {
    // outputData.value = 'zaszyfrowano przestawienia macierzowe - klucz słowny v1'
    let encryptArray = []
    let arrayToEncrypt = getText()

    let numberOfLetter = 0;
    encryptArray[0] = new Array(key.value.length)
    let j = 0

    for (; ;) {
      for (let i = 0; i < key.value.length; i++) {
        if (numberOfLetter < arrayToEncrypt.length) encryptArray[j][i] = arrayToEncrypt[numberOfLetter++]
        else encryptArray[j][i] = ''
      }
      if (numberOfLetter === arrayToEncrypt.length) break
      encryptArray[++j] = new Array(key.value.length)
    }

    outputData.value = combineArrayColumnsToGetCipher(encryptArray)
  }

  function encryptMatrixWithWordV2() {
    outputData.value = 'zaszyfrowano przestawienia macierzowe - klucz słowny v2'
    let splitTextToEncrypt = getText()
    let orderedKey = getAlphabetOrder()

    let numberOfLetter = 0;
    let encryptArray = []
    encryptArray[0] = new Array(key.value.length)
    let j = 0

    for (; ;) {
      let isLastColumn = false
      for (let i = 0; i < key.value.length; i++) {
        if (!isLastColumn) {
          encryptArray[j][i] = splitTextToEncrypt[numberOfLetter++]
          if (j + 1 === orderedKey[i] || numberOfLetter === splitTextToEncrypt.length) isLastColumn = true
        } else encryptArray[j][i] = ''
      }

      if (numberOfLetter === splitTextToEncrypt.length) break
      encryptArray[++j] = new Array(key.value.length)
      console.log("still here")
    }

    console.log(encryptArray)
    outputData.value = combineArrayColumnsToGetCipher(encryptArray)
  }


  function combineArrayColumnsToGetCipher(encryptArray) {
    let orderedKey = getAlphabetOrder()
    let encryptString = []

    for (let i = 0; i < key.value.length; i++) {
      let column = orderedKey.indexOf(i + 1)
      for (let k = 0; k < encryptArray.length; k++) {
        encryptString.push(encryptArray[k][column])
        console.log("columna" + column + " i wartość" + encryptArray[k][column])
      }
      encryptString.push(" ")
    }

    return encryptString.join("")
  }

  function getText() {
    let withoutSpace = inputData.value.split(' ')
    let textToEncrypt = []

    withoutSpace.forEach((v) => {
      textToEncrypt.push(v)
    })

    return textToEncrypt.join("").split('')
  }

  function getAlphabetOrder() {
    let keyString = key.value.split("")

    let alphabetOrderIndex = 0
    let alphabetOrder = []

    alphabet.forEach((letter) => {
      keyString.forEach((v, i) => {
        if (letter === v.toUpperCase()) alphabetOrder[i] = ++alphabetOrderIndex
      })
    })

    return alphabetOrder
  }

  function encryptCaesarCipher() {
    let n = 26
    let k = parseInt(key.value)
    let result = ''
    for (let i = 0; i < inputData.value.length; i++) {
      if (inputData.value[i] !== ' ') {
        result += alphabet[(alphabet.indexOf(inputData.value[i].toUpperCase()) + 1 + k) % n - 1]
      } else {
        result += ' '
      }
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
    let n = parseInt(key.value)
    let result = ''
    let x = new Array(n)
    for (let i = 0; i < n; i++) {
      x[i] = new Array(inputData.value.length)
    }

    for (let i = 0; i < n; i++) {
      for (let j = 0; j < inputData.value.length; j++) {
        x[i][j] = 0
      }
    }

    let rowNr
    let prevRowNr
    let step = n * 2 - 1 - 1
    if (step > n) {
      rowNr = n - (step - n) - 1
      prevRowNr = rowNr + 1
    } else {
      rowNr = step - 1
      prevRowNr = rowNr - 1
    }

    for (let i = inputData.value.length - 1; i >= 0; i--) {
      x[rowNr][i] = inputData.value[i]
      if (rowNr === 0) {
        prevRowNr = rowNr
        rowNr++
      } else if (rowNr === (n-1)) {
        prevRowNr = rowNr
        rowNr--
      } else if (prevRowNr < rowNr){
        prevRowNr = rowNr
        rowNr++
      } else {
        prevRowNr = rowNr
        rowNr--
      }
    }

    result += rowNr
    outputData.value = result
  }

  function decryptMatrixWithNumbers() {
    outputData.value = 'przestawienia macierzowe - klucz liczbowy'
  }

  function decryptMatrixWithWordV1() {
    let textToDecrypt = inputData.value.split(' ');
    let amountOfRows = 0

    textToDecrypt.forEach((v) => {
      if (v.length > amountOfRows) amountOfRows = v.length
    })

    let orderedKey = getAlphabetOrder()
    let encryptArray = []

    for (let i = 0; i < amountOfRows; i++) {
      encryptArray[i] = new Array(orderedKey.length)
      encryptArray[i].forEach((v) => {
        v = ''
      })
    }

    for (let i = 0; i < key.value.length; i++) {
      let column = orderedKey.indexOf(i + 1)
      let columnString = textToDecrypt[i].split('')

      for (let k = 0; k < amountOfRows; k++) {

        encryptArray[k][column] = columnString[k]
      }
    }

    console.log(encryptArray)
    let decryptString = []
    encryptArray.forEach((v) => {
      v.forEach((value) => {
        decryptString.push(value)
      })
    })
    console.log(decryptString)
    outputData.value = decryptString.join("")
  }

  function decryptMatrixWithWordV2() {
    let textToDecrypt = inputData.value.split(' '); // splitujemy po spacji
    let amountOfRows = 0  // inicjalizujemy ilość wierszy
    let orderedKey = getAlphabetOrder() // otrzymujemy posortowaną alfabetycznie tablicę

    textToDecrypt.forEach((v) => {
      if (v.length > amountOfRows) amountOfRows = v.length //wyznaczamy ile wierszy będzie miec nasza dwuwymiarowa tablica
    })

    let encryptArray = []
    console.log("text do dekryptowania" + textToDecrypt)

    for (let i = 0; i < amountOfRows; i++) {
      encryptArray[i] = new Array(orderedKey.length) //inicjalizujemy naszą dwuwymiarową tablicę
      encryptArray[i].forEach((v) => v = '')
    }

    let amountOfSkippedColumns = 0
    if (textToDecrypt.length < orderedKey.length) { //jeśli długość tablicy tekstu jest mniejsza od długości klucza
      let lastIndex = textToDecrypt.length - 1 //oznacza to ze jakas kolumna została nie wykorzystana przy enkryptowaniu
      for (let i = lastIndex; i < orderedKey.length - 1; i++) {
        ++amountOfSkippedColumns
        textToDecrypt.push('')
      }
    }

    console.log("teraz textToDecrypt(posplitowana po spacji tablica")
    console.log(textToDecrypt)

    let currentNumber = 1
    for (let i = 0; i < orderedKey.length; i++) {
      let column = orderedKey.indexOf(i + currentNumber)
      // let column
      // if (nextIterationRepeat) {
      //   column = orderedKey.indexOf(i + 1) // pobieramy indeks aktualnej kolumny
      // } else {
      //   column = orderedKey.indexOf(i + 2)
      //   nextIterationRepeat = true
      // }
      //
      if (orderedKey.length - i <= amountOfSkippedColumns) {
        column = orderedKey.indexOf(orderedKey[orderedKey.indexOf(i + 1)] + 1)
        ++currentNumber
      }

      let currentColumn = textToDecrypt[i].split('') // splitujemy aktualną kolumne po każdym znaku
      let numberOfLetter = 0 //numer litery
      for (let j = 0; j < amountOfRows; j++) { //iterujemy po każdym wierszu
        console.log(currentColumn[j])
        if (hasGreaterOrEqualNumberInNextIterations(orderedKey, j + 1, column) && numberOfLetter < currentColumn.length) encryptArray[j][column] = currentColumn[numberOfLetter++] //jeśli dla aktualnego wiersza
      }
    }

    console.log(encryptArray)
    let decryptString = []
    encryptArray.forEach((v) => {
      v.forEach((value) => {
        decryptString.push(value)
      })
    })
    console.log(decryptString)
    outputData.value = decryptString.join("")
  }

  function hasGreaterOrEqualNumberInNextIterations(orderedKey, currentRow, column) {
    for (let i = column; i < orderedKey.length; i++) {
      if (orderedKey[i] === currentRow) return true
    }

    return false
  }

  function decryptCaesarCipher() {
    let n = 26
    let k = parseInt(key.value)
    let result = ''
    for (let i = 0; i < inputData.value.length; i++) {
      if (inputData.value[i] !== ' ') {
        result += alphabet[(alphabet.indexOf(inputData.value[i].toUpperCase()) + 1 + n - k) % n - 1]
      } else {
        result += ' '
      }
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
