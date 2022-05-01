<template>
  <div>
    <h1>都道府県</h1>

    <div class="prefecture">
      <label v-for="prefecture in st.prefectures" :key="prefecture.prefCode">
        <input
          type="checkbox"
          :id="prefecture.prefCode"
          :value="prefecture.prefCode"
          @click="renderGraph(prefecture.prefCode, $event.target.checked)"
        />
        {{ prefecture.prefName }}
        {{ prefecture.prefCode }}
      </label>
    </div>
    <highcharts :options="st.chartOptions"></highcharts>
  </div>
</template>

<script>
import { onMounted, reactive } from 'vue';
import axios from 'axios';
import { Chart } from 'highcharts-vue';
// import Highcharts from 'highcharts';
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

    const getyears = async () => {
      st.res = await axios.get(
        url + '/population/composition/perYear?cityCode=-&prefCode=1',
        {
          headers: { 'X-API-KEY': api_key }
        }
      );
      for (let i = 0; i < st.res.data.result.data[0].data.length; i++) {
        st.years.push(st.res.data.result.data[0].data[i].year);
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

        // isChecked(checkedpref, checked);
        console.log(checkedpref);

        console.log(st.chartOptions);
        console.log(st.years);
      } else {
        // isChecked(checkedpref, checked);
        removeSeries(checkedpref);
        console.log(st.chartOptions.series);
      }
    };

    const addSeries = checkedpref => {
      let popu = [];

      getPopulation(checkedpref, popu);
      st.chartOptions.series.push({
        name: st.prefectures[checkedpref - 1].prefName,
        data: popu
      });
    };
    const removeSeries = checkedpref => {
      st.chartOptions.series = st.chartOptions.series.filter(
        series => series.name !== st.prefectures[checkedpref - 1].prefName
      );
    };

    // const isChecked = (checkedpref, checked) => {
    //   if (checked) {
    //     st.checkedpref.push(checkedpref);
    //   } else {
    //     st.checkedpref = st.checkedpref.filter(pref => pref !== checkedpref);
    //     st.populations = null;
    //   }
    // };

    st.chartOptions = {
      series: [
        {
          name: '都道府県',
          data: []
        }
      ],
      type: 'line',
      title: {
        text: 'Population composition per year'
      },
      legend: {
        align: 'right',
        verticalAlign: 'top',
        layout: 'vertical'
      },
      xAxis: {
        title: {
          text: '年度'
        },
        categories: st.years
      },
      yAxis: { title: { text: '人口数' } },
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
      logger,
      getPopulation,
      // isChecked,

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
  justify-content: space-between;
}
</style>
