<template>
  <v-row justify="center" align="center">
    <v-col cols="12" sm="8" md="6">
      <chart :data="barChartData" :options="barChartOptions" :height="200" />
    </v-col>
  </v-row>
</template>

<script>
import _ from 'lodash';    
import Chart from '~/components/Chart'

const chartColors = {
  red: 'rgb(255, 99, 132)',
  green: 'rgb(75, 192, 192)'
};

export default {
  name: 'Home',
  data() {
    return {
      barChartData: {
        labels: this.lables,
        datasets: [
          {
            label: 'Completed',
            backgroundColor: chartColors.green,
            data: null
          },
          {
            label: 'NotCompleted',
            backgroundColor: chartColors.red,
            data: null
          }
        ]
      },
      barChartOptions: {
        responsive: true,
        legend: {
          display: true,
        },
        title: {
          display: true,
          text: 'User Tasks Status'
        },
        scales: {
          xAxes: [{
            stacked: true,
          }],
          yAxes: [{
            stacked: true
          }]
        }
      },
      tasksStatus: [],
      users: [],
      userTasks: [],
      completed : [],
      notCompleted : [],
      labels: [],
    }
  },
  async fetch() {
    try {
      this.tasksStatus = await this.$axios.$get('https://mocki.io/v1/932711ca-96f8-453d-90e6-dd114ac25c25');
      this.users = await this.$axios.$get('https://mocki.io/v1/429711e1-59f6-4910-96bb-4e390c6b0063');

      this.userTasks = _.groupBy(this.tasksStatus, item => `${item.userId}-${item.completed}`);

      _.forEach(Object.keys(this.userTasks), (item) => { 
        if(item.includes('false')) {
          this.notCompleted.push(this.userTasks[item].length);
        } else {
          this.completed.push(this.userTasks[item].length);
        }

        let name = this.users.find( (user) => user.id == item.split('-')[0] ).name;

        if(this.labels.indexOf( name ) === -1) this.labels.push( name )

        this.barChartData.labels = this.labels;

      });

      this.barChartData.datasets[0].data = this.completed;
      this.barChartData.datasets[1].data = this.notCompleted;

    } catch(e) {
      this.$nuxt.error({status: 500, message: e})
    }
  },
  components: {
    Chart
  }
}
</script>