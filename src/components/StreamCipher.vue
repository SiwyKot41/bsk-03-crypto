<template>
  <div>
    <div id="container1">
      <div id="container2">
        <div class="data">
          <p class="label">Podaj wielomian:</p>
          <input placeholder="np.: 1 + x + x^2 + x^4" v-model="inputData"/>
        </div>
        <div class="errMsg"> {{ errorMessageInput }}</div>

        <div id="container3">
          <div class="data">
            <p class="label">Wybierz plik:</p>
            <input class="fileInput" type="file" @change="onFileSelected"/>
          </div>
          <p></p>
          <div class="errMsg"> {{ errorMessageNoFileInput }}</div>

          <div class="buttonContainer">
            <button v-if="!generating" @click="encrypt(true)">Szyfruj</button>
            <button v-if="!generating" @click="encrypt(false)">Deszyfruj</button>
            <!--          <button v-else @click="generating = false">Zatrzymaj generowanie</button>-->
          </div>

          <div class="data">
            <p v-if="isEncrypted" class="label">Wynik:</p>
          </div>

          <div v-if="isEncrypted" class="data">
            <a  v-if="downloadFileName === 'encryptedFile.txt'" href="#" @click.prevent="downloadItem()">Pobierz zaszyfrowany plik</a>
            <a  v-else-if="downloadFileName === 'decryptedFile.txt'" href="#" @click.prevent="downloadItem()">Pobierz odszyfrowany plik</a>
          </div>
      </div>
      </div>

    </div>
  </div>

</template>


<script setup>
import {ref} from 'vue'

const inputData = ref('')
const errorMessageInput = ref('')
const errorMessageNoFileInput = ref('')
const outputData = ref('')
const details = ref(false)
const steps = ref('')
const generating = ref(false)
const selectedFile = ref(null)
const periodResult = ref('')
const isEncrypted = ref(false)
const downloadFile = ref(null)
const downloadFileName = ref('')

// function showDetails() {
//   details.value = !details.value
// }

async function encrypt(isEncrypt) {
  outputData.value = ''
  periodResult.value = ''
  steps.value = ''

  //Sprawdzenie poprawności składowych
  errorMessageInput.value = ''
  const reg = /^[ ]+$/
  if (inputData.value.length === 0 || reg.test(inputData.value)) {
    errorMessageInput.value = "Pole ze słowem wejściowym nie może być puste";
    return false;
  }

  // usuwanie spacji z wielomianu:
  do {
    inputData.value = inputData.value.replace(" ", "")
  } while (inputData.value.includes(" "))

  // oddzielanie składowych wielomianu:
  const element = inputData.value.split('+').join('|').split('-').join('|').split('|')

  //Sprawdzenie poprawności składowych
  const reg2 = /^[0-9]+$/;    // jest sama liczba
  const reg3 = /^x+$/         // jest sam x
  const reg4 = /^[x][^][0-9]+$/   // jest x^którejś
  const reg5 = /^[0-9]+(\.[0-9]+)?$/   // jest liczba rzeczywista oddzielona kropką
  const reg6 = /^[0-9]+(\,[0-9]+)?$/   // jest liczba rzeczywista oddzielona przecinkiem
  const reg7 = /^[0-9][x][^][0-9]+$/   // jest liczba x^którejś
  const reg8 = /^[0-9][x]+$/   // jest liczba x

  for (let i = 0; i < element.length; i++) {
    if (!reg2.test(element[i]) && !reg3.test(element[i]) && !reg4.test(element[i]) && !reg5.test(element[i]) && !reg6.test(element[i])) {
      errorMessageInput.value = "Wielomian jest błędnie wpisany";
      return false;
    }
  }

  // szukamy najwyższej potęgi n tego wielomianu:
  let n = 0;
  for (let i = 0; i < element.length; i++) {
    // jeśli składowa to poprostu x -> potęga = 1
    if (reg3.test(element[i])) {
      if (n < 1) {
        n = 1;
      }
    }
    // jeśli składowa to x^a -> potęga = a
    else if (reg4.test(element[i])) {
      if (n < element[i].split('^')[1]) {
        n = element[i].split('^')[1];
      }
    }
  }

  // jeśli n = 0 to generator nie ma sensu -> zwraca info o błędnym wielomianie
  if (n === 0) {
    errorMessageInput.value = "Wielomian jest błędnie wpisany";
    return false;
  }

  // tworzymy pomocniczą tablicę potęg wypełnioną 0
  let powers = new Array(n)
  for (let i = 0; i < n; i++) {
    powers[i] = 0;
  }

  // oznaczami wartością 1 te bity, które będą poddawane operacji XOR:
  for (let i = 0; i < element.length; i++) {
    // jeśli składowa to x -> potęga = 1
    if (reg3.test(element[i])) {
      // upewniamy się że składowa o takiej potędze już nie istenieje
      if (powers[0] !== 1) {
        powers[0] = 1;
      } else {
        errorMessageInput.value = "Wielomian jest błędnie wpisany";
        return false;
      }
    }
    // jeśli składowa to x^n -> potęga = n
    else if (reg4.test(element[i])) {
      let pot = element[i].split('^')[1]
      // upewniamy się że składowa o takiej potędze już nie istenieje
      if (powers[pot-1] !== 1) {
        powers[pot-1] = 1;
      } else {
        errorMessageInput.value = "Wielomian jest błędnie wpisany";
        return false;
      }
    }
  }

  // tworzymy tablicę lfsr gdzie liczba wierszy = okres generowanego ciągu, a liczba kolumn to n
  let period = Math.pow(2, n) - 1
  let lfsr = new Array(period);
  for (let i = 0; i < period; i++) {
    lfsr[i] = new Array(n)
  }

  // wypełniamy pierwszy wiersz lfsr losowym ciągiem złożonym z 0 i 1
  for (let i = 0; i < n; i++) {
    // lfsr[0][i] = getRandomIntInclusive(0, 1);
    lfsr[0][i] = i % 2 === 0 ? 0 : 1
  }

  let p = null
  let q = null

  // tablica przechowująca cały okres wygenerowanego klucza
  let result = new Array(period)

  for (let j = 0; j < period; j++) {

    // jeśli to już ostatni wiersz, podajemy ostatni bit na wynik i kończymy
    if (j === period - 1) {
      result[j] = (lfsr[j][n-1])
    } else {
      for (let i = 0; i < n; i++) {

        // sprawdzamy czy dany bit uwzględniamy w operacji xor
        if (powers[i] === 1) {
          // jeśli to pierwszy dotąd bit uwzględniany w operacji xor zapamiętujemy go
          if (p === null) {
            p = lfsr[j][i];
          } else {
            // jeśli to kolejny bit uwzględniany w operacji xor, wykonujemy tą operację i wynik zapamiętujemy w zmiennej p
            q = lfsr[j][i]
            p = xor(p, q)
          }
        }

        // jeśli to nieostatni bit, to jego wartość przepisujemy do komórki w kolejnym wierszu, do kolumny o nr o 1 większej
        if (i !== (n-1)) {
          lfsr[j+1][i+1] = lfsr[j][i]
        } else {
          // jeśli to ostatni bit -> dodajemy go do wygenerowanego klucza
          result[j] = (lfsr[j][i])
        }
      }

      // jeśli w operacji xor mieliśmy uwzględnić tylko jeden bit -> za drugi bit do uwzględnienia z założenia przyjmujemy 0
      if (q === null) {
        p = xor(p, 0)
      }

      // wartość operacji xor wstawiamy w kolejnym wierszu w pierwszej kolumnie
      lfsr[j+1][0] = p
      p = null
      q = null
    }
  }

  console.log("nasz result")
  console.log(result)
  result.forEach((v) => {
    periodResult.value = "" + periodResult.value + v
  })


  if (selectedFile.value === null) {
    errorMessageNoFileInput.value = "Nie wybrano pliku";
    return
  }

  if (isEncrypt) downloadFileName.value = 'encryptedFile.txt'
  else downloadFileName.value = 'decryptedFile.txt'

  bitCounter = 0
  const encrypted = await encryptFile()
  isEncrypted.value = true
  downloadFile.value = new Blob([encrypted], {type: 'text/plain'})

  selectedFile.value = null
}

// function getRandomIntInclusive(min, max) {
//   min = Math.ceil(min);
//   max = Math.floor(max);
//   return Math.floor(Math.random() * (max - min + 1)) + min;
// }

function xor(p, q) {
  if (p === q) return 0;
  else return 1;
  // if ((p === 0 && q === 0) || (p === 1 && q === 1)) return 0
  // else if ((p === 0 && q === 1) || (p === 1 && q === 0)) return 1
  // else if (p === -1 && q === -1) return -1
  // else if (p === -1) return p
  // else return q
}

function onFileSelected(event) {
  selectedFile.value = event.target.files[0]
  console.log(selectedFile.value)

}

let bitCounter = 0

const getKeyBit = () => {
  if (bitCounter === periodResult.value.length || bitCounter === 0) bitCounter = 0
  return periodResult.value[bitCounter++]
}

const getKeyByte = () => {
  const byte = []

  for (let i = 0; i < 8; ++i) {
    byte.push(getKeyBit())
  }

  console.log(byte)
  console.log(periodResult.value)
  return parseInt(byte.join(''), 2)
}

const encryptFile = () => {
  const fileReader = new FileReader()
  fileReader.readAsArrayBuffer(selectedFile.value)

  return new Promise(resolve => {
    fileReader.onload = () => {
      const buffer = new Uint8Array(fileReader.result)

      // let toSave = new TextDecoder().decode(buffer);
      // console.log(toSave)
      resolve(buffer.map(byte => {
        return byte ^ getKeyByte()
      }))
    }
  })
}

async function downloadItem() {
  const link = document.createElement("a");
  link.href = URL.createObjectURL(downloadFile.value);
  link.download = downloadFileName.value
  link.click();
  URL.revokeObjectURL(link.href);
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