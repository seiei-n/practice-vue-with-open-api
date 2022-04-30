<template>
  <div>
    <h1>都道府県</h1>

    <div class="prefecture">
      <label v-for="prefecture in st.prefectures" :key="prefecture.prefCode">
        <input
          type="checkbox"
          :id="prefecture.prefCode"
          :value="prefecture.prefCode"
          @click="isChecked(prefecture.prefCode, $event.target.checked)"
        />
        {{ prefecture.prefName }}
        {{ prefecture.prefCode }}
      </label>
    </div>
    <vue-highcharts :options="chartOptions" />
  </div>
</template>

<script>
import VueHighcharts from 'vue3-highcharts';
import { defineComponent, onMounted, reactive } from 'vue';
import axios from 'axios';
export default defineComponent({
  components: {
    VueHighcharts
  },
  setup() {
    onMounted(() => {
      getPref();
    });
    const url = process.env.VUE_APP_URL;
    const api_key = process.env.VUE_APP_API_KEY;
    const st = reactive({
      prefectures: null,
      prefname: null,
      prefcode: null,
      checkedpref: [],
      populations: null,
      checked: false
    });
    const logger = prefcode => {
      console.log(prefcode);
      getPopulation(prefcode);
      console.log(st.populations);
    };

    const getPref = async () => {
      st.res = await axios.get(url + '/prefectures', {
        headers: { 'X-API-KEY': api_key }
      });
      st.prefectures = st.res.data.result;
    };

    const getPopulation = async prefcode => {
      const res = await axios.get(
        url + '/population/composition/perYear?cityCode=-&prefCode=' + prefcode,

        {
          headers: { 'X-API-KEY': api_key }
        }
      );
      st.populations = res.data.result.data[0].data[0].value;
    };

    const isChecked = (checkedpref, checked) => {
      if (checked) {
        st.checkedpref.push(checkedpref);
      } else {
        st.checkedpref = st.checkedpref.filter(pref => pref !== checkedpref);
      }

      console.log(st.checkedpref);
      console.log(checked);
    };

    const renderGraph = () => {};

    const chartOptions = {};
    return {
      st,
      logger,
      getPopulation,
      isChecked,
      renderGraph,
      chartOptions
    };
  }
});
</script>
