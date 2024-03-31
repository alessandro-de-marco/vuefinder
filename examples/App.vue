<template>
  <div class="wrapper">
    <div style="font-weight: bold;padding: 10px">External select example</div>
    <vue-finder
      v-if="files.length"
      id="my_vuefinder"
      theme="light"
      :items="files"
      :breadcrumb="breadcrumb()"
      :select-button="handleSelectButton"
      @vf-double-click="getFiles($event)"
      @vf-new-folder="console.log('new folder')"
      @vf-new-file="console.log('new file')"
      @vf-delete="console.log('Deleting...', $event)"
      @vf-rename="console.log('Renaming...', $event)"
      @vf-home="console.log($event)"
      @vf-refresh="console.log($event)"
      @vf-level-up="console.log('level up')"
      @vf-upload="console.log('Upload')"
      :owner="[
        { name: 'user', class: 'fill-sky-500 stroke-sky-500' },
        { name: 'system', class: 'fill-neutral-500 stroke-neutral-500' },
      ]"
    >
    </vue-finder>

    <button class="btn" @click="handleButton" :disabled="!selectedFiles.length">Show Selected  ({{ selectedFiles.length ?? 0 }} selected)</button>

    <div v-show="selectedFiles.length">
      <h3>Selected Files ({{ selectedFiles.length }} selected)</h3>
      <ul>
        <li v-for="file in selectedFiles" :key="file.path">
          {{ file }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import { FEATURES, FEATURE_ALL_NAMES } from '../src/components/features.js';
import title_shorten from "../src/utils/title_shorten.js";
const ENV = import.meta.env;
import axios from "axios";
const files = ref([]);

const api = axios.create({
  baseURL: ENV.VITE_BASE_URL ?? 'https://127.0.0.1:8000',
  headers: {
    Authorization: 'Bearer ' + ENV.VITE_TOKEN
  }
})

const getRootFiles = async () => {
  try {
    const response = await api.get('/api/files?filter[user]=dbc89b8d-d576-47be-93c9-c406803beaa8');
    files.value = response.data.data.map((item) => {
      return {
        id: item.data.id,
        ...item.data.attributes,
        created_at: item.data.meta.created_at,
        children: item.meta.total ?? null,
      }
    });
  } catch (error) {
    console.error(error);
  }
}

const getFiles = async (item) => {
  try {
    const response = await api.get(`/api/files/${item.id}`);
    files.value = response.data.data.map((item) => {
      return {
        id: item.data.id,
        ...item.data.attributes,
        created_at: item.data.meta.created_at,
        children: item.meta.total ?? null,
      }
    });
    app.loading = false;
  } catch (error) {
    console.error(error);
  }
}

const breadcrumb = () => {
  return [
    {
      path: '/images',
      name: 'Images'
    },
    {
      path: '/videos',
      name: 'Videos'
    },
  ]
}

getRootFiles();

const test = (item) => {
  console.log(item)
}

/** @type {import('../src/utils/ajax.js').RequestConfig} */
const request = {
  // ----- CHANGE ME! -----
  // [REQUIRED] Url for development server endpoint
  baseUrl: 'https://vuefinder.ozdemir.be/vuefinder',
  // ----- CHANGE ME! -----

  // Additional headers & params & body
  headers: { Authorization: 'Bearer ' + ENV.VITE_TOKEN },
  params: { uid: 'dbc89b8d-d576-47be-93c9-c406803beaa8' },

  // And/or transform request callback
  transformRequest: (req) => {
    return req;
  },

  // XSRF Token header name
  xsrfHeaderName: "CSRF-TOKEN",
}
const maxFileSize = ref("500MB")

const features = [
  ...FEATURE_ALL_NAMES,
  // Or remove the line above, specify the features want to include
  // Like...
  //FEATURES.LANGUAGE,
]

const selectedFiles = ref([]);

// an example how to show selected files, outside of vuefinder
// you can create a ref object and assign the items to it,
// then with a button click, you can get the selected items easily

const handleSelectButton = {
  // show select button
  active: true,
  // allow multiple selection
  multiple: false,
  // handle click event
  click: (items, event) => {
    if (!items.length) {
      alert('No item selected');
      return;
    }
    alert('Selected: ' + items[0].path);
    console.log(items, event);
  }
}

</script>

<style>
body {
  margin: 0;
  background: #eeeeee;
}
.wrapper {
  max-width: 800px;
  margin: 80px auto;
}
.btn{
  display: block;
  margin: 20px auto;
  padding: 10px 20px;
  border: 1px solid #ccc;
  border-radius: 5px;
  background: #fff;
  cursor: pointer;
  outline: none;
}
</style>
