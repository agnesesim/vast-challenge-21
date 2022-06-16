<template>
<b-container class="bv-example-row">
  <b-row class="m-5">
    <b-col sm="12" md="4">
      <b-form-group
        id="fieldset-1"
        label="Location"
        label-for="input-1"
        >
        <b-form-select id="input-1" v-model="selectedLoc" :options="locations"></b-form-select>
      </b-form-group>
    </b-col>
    <b-col sm="12" md="4">
      <b-form-group
        id="fieldset-2"
        label="Day of the Week"
        label-for="input-2"
        >
        <b-form-select id="input-2" v-model="selectedDayWeek" :options="dayWeek"></b-form-select>
      </b-form-group>
    </b-col>
    <b-col sm="12" md="4">
       <b-form-group
        id="fieldset-3"
        label="Specific day"
        label-for="input-3"
        >
        <b-form-select id="input-3" v-model="selectedDay" :options="days"></b-form-select>
      </b-form-group>
    </b-col>
  </b-row>
  <b-row>
    <b-col sm="12" md="6">
      <BarChartD3 :data_source="numbers" :options="opts"></BarChartD3>
    </b-col>
    <b-col sm="12" md="6">
      <BarChartD3 :data_source="numbers" :options="opts"></BarChartD3>
    </b-col>
  </b-row>
  <b-row>
    <b-col col lg="12" sm="12">
      <TableEmployees></TableEmployees>
    </b-col>
  </b-row>
</b-container>
</template>


<script>
  import BarChartD3 from "@/components/BarChartD3";
  import TableEmployees from "@/components/TableEmployees.vue";
  const d3 = require('d3');


export default {
  name: 'HomePage',
  components:{
    BarChartD3,
    TableEmployees,
  },
  data: function(){
    return{
      numbers:[
        {letter: 'A', frequency: 70},
        {letter: 'B', frequency: 3},
        {letter: 'C', frequency: 21},
        {letter: 'D', frequency: 6},
      ],
      opts: {
                x: d => d.letter,
                y: d => d.frequency,
                yLabel: "↑ Frequency",
                xLabel: "Letter",
                width: 600,
                height: 400,
                color: "teal"
            },
      selectedLoc: 0,
      locations:[],
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
      days:[],
    }
  },
  mounted() {
      d3.csv("/data/location.csv")
        .then((rows) => {
          var locs = []
          for(var i = 0; i < rows.length; i++){
              var l = {
                value: rows[i].id,
                text: rows[i].name,
              }
              locs.push(l);
          }
          this.locations = locs;
        });
        
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
  },
  methods: {
    // shuffleNumbers: function(){
    //   this.numbers = d3
    //   .range(Math.round(Math.random()*20))
    //   .map(()=> Math.round(Math.random()*100))
    // }
  },
}
</script>


