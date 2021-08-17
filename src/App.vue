<template>
  <div id="app">
    <line-chart :width="1000" :height="300" :chartdata="chartdata" :options="options" />
  </div>
</template>

<script>
  import LineChart from './components/Chart.vue';
  import csvFile from './assets/base_prices_one.csv';
  // import csvFile from './assets/merged.csv';
  import * as d3 from "d3";

  export default {
    name: 'App',
    components: {
      LineChart
    },
    data() {
      return {
        chartdata: {
          labels: [],
          datasets: []
        },
        options: {
          // responsive: false,
          // maintainAspectRatio: false
        },
        item: csvFile
      }
    },
    methods: {
      async parseCsv() {
        const data = await d3.csv(this.item);

        const filteredData = data.filter(el => el !== el['columns']);

        let datasets = [];
        let chartLabels = [];
        
        for (const data of filteredData) {
          let obj = {};

          let label;

          if (data.variant !== '-') {
            label = `${data.vendor} | ${data['product_name']}_${data.variant}`;
          } else {
            label = `${data.vendor} | ${data['product_name']}`;
          }

          obj['label'] = label;

          let keys = Object.keys(data).filter(key => key.includes('price'));
          keys = keys.filter(key => key !== 'price type');
          
          let newData = [];

          for (const key of keys) {
            newData.push({
              x: Number(data[`amount${key.split('price')[1]}`]),
              y: data[key].split(',').join('.')
            });
          }

          newData = newData.sort((a, b) => (a.x > b.x) ? 1 : -1);
          newData = newData.filter(el => el.x !== 0)

          obj.data = newData;

          const amounts = Object.keys(data).filter(key => key.includes('amount'));

          for (const amount of amounts) {
            chartLabels.push(data[amount]);
          }         

          datasets.push(obj);
        }

        chartLabels = [...new Set(chartLabels)].filter(el => el !== '').map(el => Number(el));
        this.chartdata.labels = chartLabels.sort(function(a, b) {
          return a - b;
        });

        this.chartdata.datasets = datasets.map(el => ({
          label: el.label,
          fill: false,
          borderColor: this.getRandomColor(),
          data: el.data,
          lineTension: 0
        }));
      },
      getRandomColor() {
        const letters = '0123456789ABCDEF';
        let color = '#';
        for (let i = 0; i < 6; i++) {
          color += letters[Math.floor(Math.random() * 16)];
        }
        return color;
      }
    },
    mounted() {
      this.parseCsv();
    }
  }
</script>

<style>
</style>
