<template>
  <div>
    <h1>todouhukenn</h1>
    <div v-for="prefecture in prefectures" :key="prefecture" class="prefecture">
      <input type="checkbox" v-model="checked" @change="selectPrefecture" />
      <label>{{ prefecture.prefName }}</label>
    </div>
  </div>
</template>

<script>
import { onMounted, reactive } from '@vue/runtime-core';
import axios from 'axios';
export default {
  setup() {
    const url = process.env.VUE_APP_URL;
    const api_key = process.env.VUE_APP_API_KEY;
    const state = reactive({
      res: null,
      prefectures: null,
      prefname: null,
      prefcode: null
    });

    async function fetchPref() {
      state.res = await axios.get(url + '/prefectures', {
        headers: { 'X-API-KEY': api_key }
      });
      state.prefectures = state.res.data.result;
    }

    onMounted(fetchPref);
    return state;
  }
};
</script>
