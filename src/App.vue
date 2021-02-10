<template>
  <div class="columns is-gapless">
    <div class="column is-one-quarter sideBar">
      <div class="wrapper p-2">
        <input type="file" id="uploadFile" @input="openFile" style="display: none;">
        <label for="uploadFile" class="button is-danger is-small mb-2 mr-2">
          <i class="icon-folder"></i>
        </label>
        <button class="button is-danger is-small mb-2 mr-2" @click="removeCurrent" v-if="state.current">
          <i class="icon-trash"></i>
        </button>
        <button class="button is-primary is-small mb-2 mr-2" @click="createDoc">
          <i class="icon-doc"></i>
        </button>
        <button :class="`button ${state.isChanged ? 'is-primary' : ''} is-small mb-2 mr-2`" @click="save">
          <i class="icon-cloud-upload"></i>
        </button>
        <button class="button is-info is-small mb-2 mr-2" @click="download">
          <i class="icon-cloud-download"></i>
        </button>
        <button class="button is-black is-small mb-2 mr-2" @click="enterKey">
          <i class="icon-lock"></i>
        </button>
        <div class="list">
          <div class="item block mb-2" v-for="item in state.data" :key="item.date">
            <button :class="`button ${item === state.current ? 'is-info' : ''}`" @click="open(item)">
              <small>
                <small>
                  {{ item.date.substr(0, 10) }}
                </small>
              </small>
              &nbsp;
              <strong>{{ item.title }}</strong>
            </button>
          </div>
        </div>
      </div>
    </div>
    <div class="column mainContent" v-if="state.current">
      <div class="wrapper p-2">
        <input type="text" class="input mb-2" v-model="state.current.title" @input="state.isChanged = true">
        <textarea class="textarea" v-model="state.current.content" @input="state.isChanged = true"></textarea>
      </div>
    </div>
  </div>
</template>

<script setup>
import { reactive } from 'vue'
import CryptoJS from 'crypto-js'

const state = reactive({
  data: [],
  current: null,
  key: sessionStorage.getItem('pandora-key') || null,
  isChanged: false
});

const save = () => {
  const plainText = JSON.stringify(state.data);
  const cipherText = state.key ? CryptoJS.AES.encrypt(plainText, state.key).toString() : plainText;
  localStorage.setItem('pandora-data', cipherText);
  state.isChanged = false;
  return cipherText;
}

const openData = (data, key) => {
  if (!data)
    data = '[]';

  try {
    data = JSON.parse(data);
  } catch(err){
    if (!key)
      key = window.prompt("Enter your encrypted key: ");
    try {
      data = CryptoJS.AES.decrypt(data, key).toString(CryptoJS.enc.Utf8);
      data = JSON.parse(data);
    } catch(err){
      alert('Wrong key');
      return;
    }
  }

  state.key = key;
  state.data = data;

  save();

  sessionStorage.setItem('pandora-key', key);
}

openData(localStorage.getItem('pandora-data'), state.key);

const createDoc = () => {
  state.data.unshift({
    title: 'New document',
    date: new Date(),
    content: ''
  });

  save();
}

const open = item => {
  state.current = item;
}

const enterKey = () => {
  state.key = window.prompt("Enter your encrypted key: ");
  save();
}

const removeCurrent = () => {
  const index = state.data.indexOf(state.current);

  if (index === -1)
    return;

  state.data.splice(index, 1);
  save();
}

const download = () => {
  var element = document.createElement('a');
  element.setAttribute('href', 'data:text/plain;charset=utf-8,' + encodeURIComponent(save()));
  element.setAttribute('download', `pandora-${(new Date()).toISOString().substr(0, 10)}.txt`);

  element.style.display = 'none';
  document.body.appendChild(element);

  element.click();

  document.body.removeChild(element);
}

const openFile = () => {
  const fileSelector = document.getElementById('uploadFile');
  const file = fileSelector.files[0];
  const reader = new FileReader();
  reader.addEventListener('load', (event) => {
    openData(event.target.result);
    fileSelector.value = null;
  });
  reader.readAsBinaryString(file);
}

</script>

<style scoped>
@import "https://cdn.jsdelivr.net/npm/bulma@0.9.1/css/bulma.min.css";
@import "https://cdnjs.cloudflare.com/ajax/libs/simple-line-icons/2.4.1/css/simple-line-icons.css";

.w-100 {
  width: 100%;
}

.sideBar .item button.button {
  text-overflow: ellipsis;
  display: block;
  overflow: hidden;
  width: 100%;
  text-align: left;
}

.mainContent .textarea {
  height: calc(100vh - 4rem);
  resize: none;
}
</style>