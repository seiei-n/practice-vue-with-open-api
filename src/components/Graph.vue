<template>
  <div>
    <h2>都道府県</h2>
    <div class="form-group">
      <div class="prefecture">
        <label
          v-for="prefecture in st.prefectures"
          :key="prefecture.prefCode"
          class="text"
        >
          <input
            type="checkbox"
            :id="prefecture.prefCode"
            :value="prefecture.prefCode"
            @click="renderGraph(prefecture.prefCode, $event.target.checked)"
          />
          {{ prefecture.prefName }}
        </label>
      </div>
    </div>
    <highcharts :options="st.chartOptions"></highcharts>
  </div>
</template>

<script>
import { onMounted, reactive } from 'vue';
import axios from 'axios';
import { Chart } from 'highcharts-vue';
export default {
  components: {
    highcharts: Chart
  },

  setup() {
    onMounted(() => {
      getPref();
      getyears();
    });
    const url = process.env.VUE_APP_URL;
    const api_key = process.env.VUE_APP_API_KEY;
    const st = reactive({
      prefectures: null,
      prefname: null,
      prefcode: null,
      chartOptions: [],
      checked: false,
      response: null,
      years: []
    });

    const getPref = async () => {
      st.response = await axios.get(url + '/prefectures', {
        headers: { 'X-API-KEY': api_key }
      });
      st.prefectures = st.response.data.result;
    };

    const getyears = async () => {
      st.response = await axios.get(
        url + '/population/composition/perYear?cityCode=-&prefCode=1',
        {
          headers: { 'X-API-KEY': api_key }
        }
      );
      for (let i = 0; i < st.response.data.result.data[0].data.length; i++) {
        st.years.push(st.response.data.result.data[0].data[i].year);
      }
    };

    const getPopulation = async (prefcode, populations) => {
      st.response = await axios.get(
        url + '/population/composition/perYear?cityCode=-&prefCode=' + prefcode,

        {
          headers: { 'X-API-KEY': api_key }
        }
      );
      for (let j = 0; j < st.response.data.result.data[0].data.length; j++) {
        populations.push(st.response.data.result.data[0].data[j].value);
      }
    };

    const renderGraph = (checkedpref, checked) => {
      if (checked) {
        addSeries(checkedpref);
        console.log(checkedpref);
        console.log(st.chartOptions);
      } else {
        removeSeries(checkedpref);
        console.log(st.chartOptions.series);
      }
    };

    const addSeries = async checkedpref => {
      let populations = [];

      await getPopulation(checkedpref, populations);
      st.chartOptions.series.push({
        name: st.prefectures[checkedpref - 1].prefName,
        data: populations
      });
    };
    const removeSeries = checkedpref => {
      st.chartOptions.series = st.chartOptions.series.filter(
        series => series.name !== st.prefectures[checkedpref - 1].prefName
      );
    };

    st.chartOptions = {
      series: [
        {
          name: '都道府県',
          data: []
        }
      ],
      type: 'line',
      title: {
        text: ''
      },
      legend: {
        align: 'right',
        verticalAlign: 'top',
        layout: 'vertical'
      },
      xAxis: {
        title: {
          text: '年度',
          align: 'high'
        },
        categories: st.years
      },
      yAxis: { title: { align: 'high', text: '人口数', rotation: 0 } },
      responsive: {
        rules: [
          {
            condition: {
              maxWidth: 500
            },
            chartOptions: {
              legend: {
                layout: 'horizontal',
                align: 'center',
                verticalAlign: 'bottom'
              }
            }
          }
        ]
      }
    };
    return {
      st,
      getPopulation,
      getyears,
      renderGraph
    };
  }
};
</script>

<style scoped>
.prefecture {
  display: flex;
  flex-wrap: wrap;
  margin-left: 5em;
  margin-right: 5em;
}
</style>
