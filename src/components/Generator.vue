<template>
  <div>
    <div id="container1">
      <div id="container2">
        <div class="data">
          <p class="label">Podaj wielomian:</p>
          <input placeholder="np.: 1 + x + x^2 + x^4" v-model="inputData" />
        </div>
        <div class="errMsg"> {{errorMessageInput}}</div>

        <div class="buttonContainer">
          <button @click="generate">Rozpocznij generowanie</button>
        </div>

        <div class="data">
          <p class="label">Wynik:</p>
          <textarea ref="ourResult" class="result" :readonly="true">{{outputData}}</textarea>
        </div>

        <div class="buttonContainer">
          <button @click="showDetails">Pokaż kroki</button>
        </div>
      </div>
      <div v-if="details" id="container3">
        <textarea ref="ourResult" class="steps" :readonly="true">{{outputData}}</textarea>
      </div>

    </div>
  </div>

</template>


<script setup>
import {ref} from 'vue'

const inputData = ref('')
const errorMessageInput = ref('')
const outputData = ref('')
const details = ref(false)

function showDetails() {
  details.value = !details.value
}

function generate() {
  outputData.value = ''

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


  // WYPISYWANIE POTĘG DO USUNIĘCIA
  for (let i = 0; i < n; i++) {
    outputData.value += powers[i]
  }
  outputData.value += '|'

  // tworzymy tablicę lfsr gdzie liczba wierszy = okres generowanego ciągu, a liczba kolumn to n
  let period = Math.pow(2, n) - 1
  let lfsr = new Array(period);
  for (let i = 0; i < period; i++) {
    lfsr[i] = new Array(n)
  }

  // wypełniamy pierwszy wiersz lfsr losowym ciągiem złożonym z 0 i 1
  for (let i = 0; i < n; i++) {
    lfsr[0][i] = getRandomIntInclusive(0, 1);
    outputData.value += lfsr[0][i]
  }
  // outputData.value += '| n = ' + n + "| okres = "
  // outputData.value += Math.pow(2, n) - 1
  outputData.value += '|'

  let p = null
  let q = null
  let result = new Array(period)

  for (let j = 0; j < period; j++) {
    if (j === period - 1) {
      result[j] = (lfsr[j][n-1])
    } else {
      for (let i = 0; i < n; i++) {
        if (powers[i] === 1) {
          if (p === null) {
            p = lfsr[j][i];
          } else {
            q = lfsr[j][i]
            p = xor(p, q)
          }
        }
        if (i !== (n-1)) {
          lfsr[j+1][i+1] = lfsr[j][i]
        } else {
          result[j] = (lfsr[j][i])
        }
      }
      if (q === null) {
        p = xor(p, 0)
      }
      lfsr[j+1][0] = p
      // outputData.value += "xor = " + p + "|"
      p = null
      q = null
    }
  }

  for (let i = 0; i < period; i++) {
    outputData.value += result[i]
  }
  // for (let j = 0; j < period; j++) {
  //   for (let i = 0; i < n; i++) {
  //     outputData.value += lfsr[j][i]
  //   }
  //   outputData.value += "|"
  // }

}

function getRandomIntInclusive(min, max) {
  min = Math.ceil(min);
  max = Math.floor(max);
  return Math.floor(Math.random() * (max - min + 1)) + min;
}

function xor(p, q) {
  if (p === q) return 0;
  else return 1;
  // if ((p === 0 && q === 0) || (p === 1 && q === 1)) return 0
  // else if ((p === 0 && q === 1) || (p === 1 && q === 0)) return 1
  // else if (p === -1 && q === -1) return -1
  // else if (p === -1) return p
  // else return q
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

  justify-content: center;
  align-items: center;
  position: relative;
  flex-grow: 1;
  flex-shrink: 0;
  flex-basis: auto;
  margin: 30px 0;
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
  flex: none;
  order: 1;
  flex-grow: 0;
  margin: 0px 4px;
  font-size: 30px;
}

.steps {
  height: auto;
  width: auto;
  margin: 30px;
  justify-self: center;
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
