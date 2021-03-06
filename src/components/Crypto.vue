<template>
  <div>
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
        <textarea ref="ourResult" class="result" :readonly="true">{{outputData}}</textarea>
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

    //Sprawdzenie czy input nie jest pusty
    if (inputData.value.length === 0 || reg.test(inputData.value)) {
      errorMessageInput.value = "Pole ze słowem wejściowym nie może być puste";
      return false;
    }

    //Sprawdzenie czy text składa się tylko z liter
    const reg2 = /^[a-zA-Z ]+$/;
    if (!reg2.test(inputData.value)) {
      errorMessageInput.value = "Użyto niedozwolonych znaków - używaj tylko liter";
      return false;
    }

    //Usuwanie wszystkich białych znaków ze słowa wejściowego jeśli użytkownik takowe podał
    if (selected.value !== 'Przestawienia macierzowe - klucz słowny v1' && selected.value !== 'Przestawienia macierzowe - klucz słowny v2') {
      do {
        inputData.value = inputData.value.replace(" ", "")
      } while (inputData.value.includes(" "))
    }

    return true;
  }

  function testKey() {
    errorMessageKey.value = ''

    //Sprawdzanie czy klucz nie jest pusty lub złożony z samych spacji
    const reg = /^[ ]+$/
    if (key.value.length === 0 || reg.test(key.value)) {
      errorMessageKey.value = "Pole z kluczem nie może być puste";
      return false;
    } else switch (selected.value) {
      case 'Rail Fence':

        //Sprawdzenie czy klucz jest złożony tylko z dodatnich liczb
        const reg3 = /^[0-9]+$/;
        if (!reg3.test(key.value) || key.value <= 0) {
          errorMessageKey.value = "Użyto niedozwolonych znaków - podaj tylko dodatnią liczbę";
          return false;
        }
        break;
      case 'Przestawienia macierzowe - klucz liczbowy':

        //Sprawdzenie czy klucz jest typu "liczba-liczba-...-liczba"
        const isValid = keyToTest => /^(?:\d+-)*\d+$/.test(keyToTest)
            && keyToTest.split('-')
                .map(i => +i)
                .sort((a, b) => a - b)
                .reduce((acc, a , b) => acc && a === b + 1, true)
        if (!isValid(key.value)) {
          errorMessageKey.value = "Użyto niedozwolonych znaków - używaj schematu liczba-liczba-liczba...-liczba w odpowiedniej kolejności"
          return false
        }

        break;
      case 'Przestawienia macierzowe - klucz słowny v1':

        //Sprawdzenie czy klucz jest złożony tylko z liter
        const regv1 = /^[a-zA-Z]+$/;
        if (!regv1.test(key.value)) {
          errorMessageKey.value = "Użyto niedozwolonych znaków - używaj tylko liter";
          return false;
        }
          break
      case 'Przestawienia macierzowe - klucz słowny v2':

        //Sprawdzenie czy klucz jest złożony tylko z liter
        const regv2 = /^[a-zA-Z]+$/;
        if (!regv2.test(key.value)) {
          errorMessageKey.value = "Użyto niedozwolonych znaków - używaj tylko liter";
          return false;
        }
        break;
      case 'Szyfr Cezara':

        //Sprawdzenie czy klucz składa się tylko z dodatniej liczby
        const reg4 = /^[0-9]+$/;
        if (!reg4.test(key.value)) {
          errorMessageKey.value = "Użyto niedozwolonych znaków - podaj tylko liczbę";
          return false;
        }
        break;
      case 'Szyfrowanie Vigenere’a':

        //Sprawdzenie czy klucz jest złożony tylko z liter
        const reg5 = /^[a-zA-Z]+$/;
        if (!reg5.test(key.value)) {
          errorMessageKey.value = "Użyto niedozwolonych znaków - używaj tylko liter";
          return false;
        }
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
    let n = parseInt(key.value)
    let result = ''

    // Jeśli klucz = 1 to zaszyfrowane słowo wygląda tak samo jak słowo wejściowe
    if (n === 1) {
      result = inputData.value;
    }
    else {
      let nrRow = 0;
      let nrPrevRow = -1;
      let x = new Array(n)

      // tworzę tablicę o ilości wierszy = wartość klucza oraz o ilości kolumn odpowiadającej długości słowa wejściowego
      for (let i = 0; i < n; i++) {
        x[i] = new Array(inputData.value.length)
      }

      // wypełniam tablicę zerami
      for (let i = 0; i < n; i++) {
        for (let j = 0; j < inputData.value.length; j++) {
          x[i][j] = 0
        }
      }

      // dla każdej litery słowa wejściowego poruszam się odpowiednio skos-dół/skos-góra i wstawiam daną literę w
      // odpowiedniej komórce tabeli
      for (let j = 0; j < inputData.value.length; j++) {
        x[nrRow][j] = inputData.value[j];
        if (nrRow === 0) {
          nrPrevRow = nrRow;
          nrRow++;
        } else if(nrRow === n-1) {
          nrPrevRow = nrRow;
          nrRow--;
        }
        else if (nrRow < nrPrevRow) {
          nrPrevRow = nrRow;
          nrRow--;
        } else if (nrRow > nrPrevRow) {
          nrPrevRow = nrRow;
          nrRow++;
        }
      }

      // odczytuję z każdego wiersza pokolei od lewej do prawej komórki zawierające tylko litery
      for (let i = 0; i < n; i++) {
        for (let j = 0; j < inputData.value.length; j++) {
          if (x[i][j] !== 0) {
            result += x[i][j];
          }
        }
      }
    }

    outputData.value = result;
  }

  function encryptMatrixWithNumbers() {
    // outputData.value = 'przestawienia macierzowe - klucz liczbowy'
    let keyArray = key.value.split("-")
    let textToEncrypt = inputData.value.split('')
    let numberOfLetter = 0
    let encryptArray = []

    encryptArray[0] = new Array(key.value.length)
    let j = 0

    for (; ;) {
      for (let i = 0; i < keyArray.length; i++) {
        if (numberOfLetter < textToEncrypt.length) encryptArray[j][i] = textToEncrypt[numberOfLetter++]
        else encryptArray[j][i] = ''
      }
      if (numberOfLetter === textToEncrypt.length) break
      encryptArray[++j] = new Array(keyArray.length)
    }


    let encryptString = []

    for (let row = 0; row < encryptArray.length; row++) {
      for (let i = 0; i < keyArray.length; i++) {
        encryptString.push(encryptArray[row][keyArray[i] - 1])
      }
    }

    outputData.value = encryptString.join("")
  }

  function encryptMatrixWithWordV1() {
    let encryptArray = []
    let textToEncrypt = getText()

    let numberOfLetter = 0;
    encryptArray[0] = new Array(key.value.length)
    let j = 0

    for (; ;) {
      for (let i = 0; i < key.value.length; i++) {
        if (numberOfLetter < textToEncrypt.length) encryptArray[j][i] = textToEncrypt[numberOfLetter++]
        else encryptArray[j][i] = ''
      }
      if (numberOfLetter === textToEncrypt.length) break
      encryptArray[++j] = new Array(key.value.length)
    }

    outputData.value = combineArrayColumnsToGetCipher(encryptArray)
  }

  function encryptMatrixWithWordV2() {
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

  function encryptCaesarCipher() {
    let n = 26
    let k = parseInt(key.value)
    let result = ''

    // każdą literę słowa wejściowego zastępują nową literą 'obliczoną' na podstawie wzoru
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
    let k = key.value

    let p = 0;
    if (k.length > inputData.value.length) {
      errorMessageKey.value = "Długość klucza nie może być dłuższa od długości słowa wejściowego";
    } else {
      // jeśli dł. klucza < dł. słowa wejściowego, uzupełniam brakujące litery zapętlonym kluczem aż klucz będzie miał
      // tę samą dł. co słowo wejściowe
      if (k.length < inputData.value.length) {
        while (k.length !== inputData.value.length) {
          k += key.value[p % key.value.length]
          p++
        }
      }
      let result = ''

      // tworzę tablicę odpowiedniej wielkości
      let x = new Array(26)
      for (let i = 0; i < 26; i++) {
        x[i] = new Array(26)
      }

      // i uzupełniam ją odpowiednimi literami alfabetu aby otrzymać tablicę Vigenere'a
      for (let i = 0; i < 26; i++) {
        for (let j = 0; j < 26; j++) {
          x[i][j] = alphabet[(i + j) % 26];
        }
      }

      // dla każdej literki słowa wejściowego odczytuję odpowiadającą jej literkę z tablicy Vigenere'a
      for (let i = 0; i < inputData.value.length; i++) {
        result += x[alphabet.indexOf(k[i].toUpperCase())] [alphabet.indexOf(inputData.value[i].toUpperCase())]
      }

      outputData.value = result
    }
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
    // jeśli klucz = 1 to słowo odszyfrowane jest takie samo jak słowo wejściowe
    if (n === 1) {
      result = inputData.value;
    } else {
      let nrRow = 0;
      let nrPrevRow = -1;
      let x = new Array(n)

      // tworzę tablicę o ilości wierszy = wartość klucza oraz o ilości kolumn odpowiadającej długości słowa wejściowego
      for (let i = 0; i < n; i++) {
        x[i] = new Array(inputData.value.length)
      }

      // wypełniam tablicę zerami
      for (let i = 0; i < n; i++) {
        for (let j = 0; j < inputData.value.length; j++) {
          x[i][j] = 0
        }
      }

      // dla każdej litery słowa wejściowego poruszam się odpowiednio skos-dół/skos-góra i wstawiam wartość 1 w
      // odpowiedniej komórce tabeli, aby zaznaczyć gdzie byłoby umieszczane słowo wejściowe przy szyfrowaniu
      for (let j = 0; j < inputData.value.length; j++) {
        x[nrRow][j] = 1;
        if (nrRow === 0) {
          nrPrevRow = nrRow;
          nrRow++;
        } else if(nrRow === n-1) {
          nrPrevRow = nrRow;
          nrRow--;
        }
        else if (nrRow < nrPrevRow) {
          nrPrevRow = nrRow;
          nrRow--;
        } else if (nrRow > nrPrevRow) {
          nrPrevRow = nrRow;
          nrRow++;
        }
      }

      let inputIndex = 0;

      // przechodzę kolejno każdy wiersz od lewej do prawej; jeśli wartość komórki wynosi 1, wstawiam tam kolejne litery
      // słowa wejściowego
      for (let i = 0; i < n; i++) {
        for (let j = 0; j < inputData.value.length; j++) {
          if (x[i][j] === 1) {
            x[i][j] = inputData.value[inputIndex];
            inputIndex++;
          }
        }
      }

      nrRow = 0;
      nrPrevRow = -1;

      // ponownie poruszam się odpowiednio skos-dół/skos-góra i odczytuje wpisane tam w poprzednim kroku wartości, które
      // stanowią odszyfrowane słowo
      for (let j = 0; j < inputData.value.length; j++) {
        result += x[nrRow][j];
        if (nrRow === 0) {
          nrPrevRow = nrRow;
          nrRow++;
        } else if(nrRow === n-1) {
          nrPrevRow = nrRow;
          nrRow--;
        }
        else if (nrRow < nrPrevRow) {
          nrPrevRow = nrRow;
          nrRow--;
        } else if (nrRow > nrPrevRow) {
          nrPrevRow = nrRow;
          nrRow++;
        }
      }
    }

    outputData.value = result;
  }

  function decryptMatrixWithNumbers() {
    let keyArray = key.value.split("-")
    let textToDecrypt = inputData.value.split('')
    let numberOfLetter = 0
    let encryptArray = []

    encryptArray[0] = new Array(key.value.length)
    let j = 0

    for (; ;) {
      for (let i = 0; i < keyArray.length; i++) {
        if (numberOfLetter < textToDecrypt.length) {
          if (textToDecrypt.length - numberOfLetter + 1 > keyArray[i] - i)
            encryptArray[j][keyArray[i] - 1] = textToDecrypt[numberOfLetter++]
        } else encryptArray[j][i] = ''
      }
      if (numberOfLetter === textToDecrypt.length) break
      encryptArray[++j] = new Array(keyArray.length)
    }

    let decryptString = []
    encryptArray.forEach((v) => {
      v.forEach((value) => {
        decryptString.push(value)
      })
    })

    outputData.value = decryptString.join("")
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

  for (let i = 0; i < amountOfRows; i++) {
    encryptArray[i] = new Array(orderedKey.length) //inicjalizujemy naszą dwuwymiarową tablicę
    encryptArray[i].forEach((v) => v = '')
  }

  let areSkippedColumns = false
  let amountOfSkippedColumns = 0
  if (textToDecrypt.length < orderedKey.length) { //jeśli długość tablicy tekstu jest mniejsza od długości klucza
    let lastIndex = textToDecrypt.length - 1 //oznacza to ze jakas kolumna została nie wykorzystana przy enkryptowaniu
    for (let i = lastIndex; i < orderedKey.length - 1; i++) {
      ++amountOfSkippedColumns
      areSkippedColumns = true
      textToDecrypt.push('')
    }
  }

  let currentNumber = 1
  for (let i = 0; i < orderedKey.length - (currentNumber - 1); i++) {
    let column = orderedKey.indexOf(i + currentNumber)
    while (orderedKey.length - orderedKey.indexOf(i + currentNumber) <= amountOfSkippedColumns && areSkippedColumns) {
      column = orderedKey.indexOf(orderedKey[orderedKey.indexOf(i + currentNumber)] + 1)
      ++currentNumber
    }

    let currentColumn = textToDecrypt[i].split('') // splitujemy aktualną kolumne po każdym znaku
    let numberOfLetter = 0 //numer litery
    for (let j = 0; j < amountOfRows; j++) { //iterujemy po każdym wierszu
      console.log(currentColumn[j])
      if (hasGreaterOrEqualNumberInNextIterations(orderedKey, j + 1, column) && numberOfLetter < currentColumn.length) encryptArray[j][column] = currentColumn[numberOfLetter++] //jeśli dla aktualnego wiersza
    }
  }

  let decryptString = []
  encryptArray.forEach((v) => {
    v.forEach((value) => {
      decryptString.push(value)
    })
  })

  outputData.value = decryptString.join("")
}



function decryptCaesarCipher() {
    let n = 26
    let k = parseInt(key.value)
    let result = ''

    // każdą literkę zaszyfrowanego słowa zastępują literką "obliczoną" według wzoru
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
    let k = key.value

    let p = 0;
    if (k.length > inputData.value.length) {
      errorMessageKey.value = "Długość klucza nie może być dłuższa od długości słowa wejściowego";
    } else {
      // jak w przypadku szyfrowania uzupełniam brakującą część klucza jeśli istnieje taka potrzeba
      if (k.length < inputData.value.length) {
        while (k.length !== inputData.value.length) {
          k += key.value[p % key.value.length]
          p++
        }
      }

      let result = ''

      // tworzę tablicę odpowiedniej wielkości
      let x = new Array(26)
      for (let i = 0; i < 26; i++) {
        x[i] = new Array(26)
      }

      // uzupełniam utworzoną tablicę odpowiednimi literkami alfabetu
      for (let i = 0; i < 26; i++) {
        for (let j = 0; j < 26; j++) {
          x[i][j] = alphabet[(i + j) % 26];
        }
      }

      // zastępują każdą literę zaszyfrowanego słowa odpowiednią literą odczytaną według wzoru z tablicy
      for (let i = 0; i < inputData.value.length; i++) {
        result += alphabet[x[alphabet.indexOf(k[i].toUpperCase())].indexOf(inputData.value[i].toUpperCase())]
      }

      outputData.value = result
    }
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

function hasGreaterOrEqualNumberInNextIterations(orderedKey, currentRow, column) {
  for (let i = column; i < orderedKey.length; i++) {
    if (orderedKey[i] === currentRow) return true
  }

  return false
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

</style>
