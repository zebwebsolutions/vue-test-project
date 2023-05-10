<template>
  <div>
    <canvas ref="radarChart"></canvas>
  </div>
</template>

<script>
import Chart from 'chart.js/auto';
import ChartDataLabels from 'chartjs-plugin-datalabels';

export default {
  name: 'PokemonStats',
  props: {
    stats: {
      type: Array,
      required: true,
    },
  },
  mounted() {
    this.renderChart();
  },
  methods: {
    renderChart() {
      const labels = this.stats.map(stat => stat.stat.name);
      const values = this.stats.map(stat => stat.base_stat);

      const data = {
        labels,
        datasets: [
          {
            label: 'Stats',
            data: values,
            fill: true,
            backgroundColor: 'rgba(47, 152, 251, 0.2)',
            borderColor: 'rgb(47, 152, 251)',
            pointBackgroundColor: 'rgb(47, 152, 251)',
            pointBorderColor: '#fff',
            pointHoverBackgroundColor: '#fff',
            pointHoverBorderColor: 'rgb(255, 99, 132)',
            pointLabels: labels
          },
        ],
      };

      const config = {
  type: 'radar',
  data: data,
  options: {
    scales: {
      r: {
        angleLines: {
          display: false
        },
        suggestedMin: 0,
        suggestedMax: 200,
      }
    },
    plugins: {
      datalabels: {
        display: true, // enable the plugin
        color: '#000',
        font: {
          weight: 'bold'
        },
        formatter: function(value, context) {
          const values = context.chart.data.datasets[0].data;
          const label = context.chart.data.labels[context.dataIndex];
          const index = context.dataIndex;
          return `${value} (${values[index]})`;
        },
        labels: {
          title: {
            color: '#000',
            font: {
              weight: 'bold'
            }
          }
        }
      }
    }
  }
};





      this.chart = new Chart(this.$refs.radarChart, config);
    },
  },
  beforeUnmount() {
    if (this.chart) {
      this.chart.destroy();
    }
  },
};
</script>
