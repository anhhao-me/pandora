<template>
  <div class="columns is-gapless">
    <div class="column is-one-quarter sideBar">
      <div class="wrapper p-2">
        <button class="button is-danger is-small mb-2 mr-2">Open</button>
        <button class="button is-primary is-small mb-2 mr-2" @click="createDoc">Create</button>
        <button :class="`button ${state.isChanged ? 'is-primary' : ''} is-small mb-2 mr-2`" @click="save">Save</button>
        <button class="button is-info is-small mb-2 mr-2">Download</button>
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

const state = reactive({
  data: JSON.parse(localStorage.getItem('pandora-data') || '[]'),
  current: null
});

const save = () => {
  localStorage.setItem('pandora-data', JSON.stringify(state.data));
}

const createDoc = () => {
  state.data.unshift({
    title: 'New document',
    date: new Date(),
    content: '',
    isChanged: false
  });

  save();
}

const open = item => {
  state.current = item;
}

</script>

<style scoped>
@import "https://cdn.jsdelivr.net/npm/bulma@0.9.1/css/bulma.min.css";

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