<template>
<b-container class="bv-example-row">

  <b-row class="m-3">
    <b-col sm="12">
      <b-card 
        align="center"
        style=";margin-bottom:10px;"
      >
        <div>
          <label for="range-1">Example range with min and max</label>
          <b-form-input id="range-1" v-model="selectedDay" type="range" min="0" max="5"></b-form-input>
          <div class="mt-2">Value: {{ selectedDay }}</div>
        </div>
      </b-card>
    </b-col>
  </b-row>
  <b-row class="m-3">
    <b-col sm="12" md="9">
      <b-card 
        header="Abila" 
        align="center"
        header-border-variant="info"
        header-text-variant="info"
        style=";margin-bottom:10px;"
      >
        <MapD3></MapD3>
      </b-card>
    </b-col>
    <b-col sm="12" md="3">
        <b-card no-body 
          header="Employees"
          header-border-variant="info"
          header-text-variant="info"
          align="center"
        >
        <b-list-group>
          <b-list-group-item v-for="emp in employees" :id="emp.value" :key="emp.value" class="d-flex justify-content-between align-items-center empList">
            {{emp.text}}
          </b-list-group-item>
        </b-list-group>
      </b-card>
    </b-col>
  </b-row>
</b-container>
</template>


<script>
  import MapD3 from "@/components/MapD3";


  const d3 = require('d3');

export default {
  name: 'AboutView',
  components:{
    MapD3
    // StackedBarChartD3
  },
  data: function(){
    return{
      employees: [],
      selectedDayWeek: 0,
      dayWeek:[
        {value: 0, text: 'All'},
        {value: 1, text: 'Monday'},
        {value: 2, text: 'Tuesday'},
        {value: 3, text: 'Wednesday'},
        {value: 4, text: 'Thursday'},
        {value: 5, text: 'Friday'},
        {value: 6, text: 'Saturday'},
        {value: 7, text: 'Sunday'},
      ],
      selectedDay: '01/01/1900',
      days:[]
    }
  },
  async mounted () {
    
    d3.csv("/data/specific-days.csv")
    .then((rows) => {
      var ds = []
      for(var i = 0; i < rows.length; i++){
          var d = {
            value: rows[i].day,
            text: rows[i].eu_day,
          }
          ds.push(d);
      }
      this.days = ds;
    });
    
    this.refresh();

    d3.csv("/data/car-assignments.csv")
    .then((rows) => {
      var emp = []
      for(var i = 0; i < rows.length; i++){
          var e = {
            value: rows[i].CarID == null ? i+1 : rows[i].CarID, 
            text: rows[i].LastName + ' ' + rows[i].FirstName,
          }
          emp.push(e);
      }
      this.employees = emp;
    });
  },
  methods: {
    refresh: function() {
     console.log('ok')
    }
  }
}
</script>

<style scoped>
.list-group{
    max-height: 450px;
    margin-bottom: 10px;
    overflow:scroll;
    -webkit-overflow-scrolling: touch;
    margin: 0px;
    padding:0px !important;
}

.locList .active{
  background-color: teal !important;
}
.container{
  max-width: 1600px;
}
</style>