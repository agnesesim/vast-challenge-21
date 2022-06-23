<template>
<b-container class="bv-example-row">
  <b-row class="m-2">
    <b-col sm="12" md="4">
      <b-form-group
        id="fieldset-1"
        label="Location"
        label-for="input-1"
        >
        <b-form-select id="input-1" @change="refresh" v-model="selectedLoc" :options="locations"></b-form-select>
      </b-form-group>
    </b-col>
    <b-col sm="12" md="4">
      <b-form-group
        id="fieldset-2"
        label="Day of the Week"
        label-for="input-2"
        >
        <b-form-select id="input-2"  @change="refresh" v-model="selectedDayWeek" :options="dayWeek"></b-form-select>
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
  <b-row class="m-2">
    <b-col sm="12" md="3">
      <b-card no-body 
        header="Locations"
        header-border-variant="info"
        header-text-variant="info"
        align="center"
      >
        <b-list-group>
          <b-list-group-item v-for="loc in locations" :id="loc.value" :key="loc.value" class="d-flex justify-content-between align-items-center locList" @click="changeLocation(loc.value)">
            {{loc.text}}
            <b-badge v-if="loc.pop > 150" variant="danger">very popular</b-badge>
            <b-badge v-else-if="loc.pop >= 20" variant="warning">popular</b-badge>
            <b-badge v-else-if="loc.pop < 20" variant="info">unpopular</b-badge>
            <!-- <b-badge variant="info">{{loc.pop}}</b-badge> -->
          </b-list-group-item>
        </b-list-group>
      </b-card>
    </b-col>
      <b-col sm="12" md="6">
        <b-card 
          header="Days and Hours frequented" 
          align="center"
          header-border-variant="info"
          header-text-variant="info"
        >
          <BarChartD3 :data_source="cc_freq" :options="opts"></BarChartD3>
        </b-card>
      </b-col>
      <b-col sm="12" md="3">
          <b-card no-body 
            header="Credit Card used"
            header-border-variant="danger"
            header-text-variant="danger"
            align="center"
          >
          <b-list-group>
            <b-list-group-item v-for="num in cc_number" :key="num.key" class="d-flex justify-content-between align-items-center">
              **** **** **** {{num.key}}
              <b-badge variant="info">{{num.value}}</b-badge>
            </b-list-group-item>
          </b-list-group>
        </b-card>
      </b-col>
  </b-row>
  <!-- <b-row>
    <b-col col lg="12" sm="12">
      <TableEmployees></TableEmployees>
    </b-col>
  </b-row> -->
</b-container>
</template>


<script>
  import BarChartD3 from "@/components/BarChartD3";
  // import TableEmployees from "@/components/TableEmployees.vue";
  import $ from 'jquery';
  import {nest} from 'd3-collection';

  const d3 = require('d3');

export default {
  name: 'HomePage',
  components:{
    BarChartD3,
    // TableEmployees,
  },
  data: function(){
    return{
      cc_freq:[],
      cc_number: [],
      opts: {
                x: d => d.key,
                y: d => d.value,
                width: 600,
                height: 400,
                color: "teal",
                yDomain: [0,40]
            },
      selectedLoc: 11,
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
            pop: rows[i].popularity
          }
          locs.push(l);
      }
      locs = locs.sort((a, b) => parseFloat(b.pop) - parseFloat(a.pop));
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

    this.refresh();
  },
  methods: {
    changeLocation: function(locID){
      $('.locList').removeClass('active');  
      $('#' + locID).addClass('active');

      this.selectedLoc = locID;
      this.refresh();
    },
    refreshFreq: async function(){
      var location_selected = this.selectedLoc;
      var weekday_selected = this.selectedDayWeek;

      var datasource = await d3.csv("/data/credit-cards-data.csv", function(d) {
         return {
          location_id : d.location_id,
          weekday_id : +d.weekday_id,
          weekday : d.weekday,
          hour : +d.hour,
          day : d.day
        };
      }).then(function(data){
        console.log("before-"+data.length);
        console.log("loc:" + location_selected);
        console.log("wek:" + weekday_selected);

        data = data.filter( function(d){
          if (location_selected == 0 || d["location_id"] == location_selected) 
              { 
                return d; 
              } 
        }).filter( function(d){
          if (weekday_selected == 0 || d["weekday_id"] == weekday_selected) 
              { 
                return d; 
              } 
        });

        console.log("after-"+data.length);
        var datagroup = nest()
          .key(function(d) { if (weekday_selected == 0) return d.weekday;
            else return d.hour })
          .rollup(function(v) { return v.length; })
          .entries(data);
          
        return datagroup;
      });

      var fulldata = [];
      if (weekday_selected == 0){
        fulldata = [
          {key: 'MON', value:0},
          {key: 'TUE', value:0},
          {key: 'WED', value:0},
          {key: 'THU', value:0},
          {key: 'FRI', value:0},
          {key: 'SAT', value:0},
          {key: 'SUN', value:0},
        ]
      }
      else {
        fulldata = [
          {key: 3, value:0},
          {key: 4, value:0},
          {key: 5, value:0},
          {key: 6, value:0},
          {key: 7, value:0},
          {key: 8, value:0},
          {key: 9, value:0},
          {key: 10, value:0},
          {key: 11, value:0},
          {key: 12, value:0},
          {key: 13, value:0},
          {key: 14, value:0},
          {key: 15, value:0},
          {key: 16, value:0},
          {key: 17, value:0},
          {key: 18, value:0},
          {key: 19, value:0},
          {key: 20, value:0},
          {key: 21, value:0},
          {key: 22, value:0}
        ]
      }
      datasource.forEach(function(obj){
        var objIndex = fulldata.findIndex((o => o.key == obj.key));
        fulldata[objIndex].value = obj.value;
      });
      this.cc_freq = fulldata;
    },
     refreshNum: async function(){
      var location_selected = this.selectedLoc;
      var weekday_selected = this.selectedDayWeek;

      var datasource = await d3.csv("/data/credit-cards-data.csv", function(d) {
         return {
          location_id : d.location_id,
          weekday_id : +d.weekday_id,
          day : d.day,
          number: d.last4ccnum
        };
      }).then(function(data){
        data = data.filter( function(d){
          if (location_selected == 0 || d["location_id"] == location_selected) 
              { 
                return d; 
              } 
        }).filter( function(d){
          if (weekday_selected == 0 || d["weekday_id"] == weekday_selected) 
              { 
                return d; 
              } 
        });
        var datagroup = nest()
          .key(function(d) { return d.number;})
          .rollup(function(v) { return v.length; })
          .entries(data); 
          
        datagroup = datagroup.sort((a, b) => parseFloat(b.value) - parseFloat(a.value));

        return datagroup;
      });

      this.cc_number = datasource;
    },
    refresh: function() {
      this.refreshFreq();
      this.refreshNum();
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

.locList .activa{
  background-color: teal !important;
}
.container{
  max-width: 1600px;
}
</style>