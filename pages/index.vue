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
