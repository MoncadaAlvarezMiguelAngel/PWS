<template>
  <div>
    <ApexChart v-if="ready" :options="chartOptions" :series="series"></ApexChart>
  </div>
</template>

<script>
import ApexChart from 'vue3-apexcharts';

export default {
  name: 'GanttChart',
  components: { ApexChart },
  data() {
    return {
      ready: false,
      chartOptions: {
        chart: {
          type: 'rangeBar',
        },
        plotOptions: {
          bar: {
            horizontal: true,
          },
        },
        xaxis: {
          type: 'datetime',
        },
        fill: {
          type: 'gradient',
        },
      },
      series: [
        {
          name: 'Tasks',
          data: [],
        },
      ],
    };
  },
  mounted() {
    fetch('/task?minProjects=1&limit=10', {
      method: 'GET',
    })
      .then((res) => {
        res.json()
          .then((data) => {
            this.series[0].data = data.map((task) => ({
              x: task.name,
              y: task.name,
              fillColor: '#008FFB',
            }));
            this.ready = true;
          })
          .catch((err) => console.error(err.message));
      })
      .catch((err) => console.error(err.message));
  },
};
</script>
