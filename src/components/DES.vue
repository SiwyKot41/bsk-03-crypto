<template>
  <div>
        <div id="container1">
          <div class="data">
            <p class="label">Wybierz plik:</p>
            <input class="fileInput" type="file" @change="onFileSelected"/>
          </div>
          <p></p>
          <div class="errMsg"> {{ errorMessageNoFileInput }}</div>
    </div>
  </div>

</template>


<script setup>
import {ref} from 'vue'

const errorMessageInput = ref('')
const errorMessageNoFileInput = ref('')

function onFileSelected(event) {
  selectedFile.value = event.target.files[0]
  console.log(selectedFile.value)

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