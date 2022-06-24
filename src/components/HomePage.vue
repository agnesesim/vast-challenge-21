<template>
<b-container class="bv-example-row">
  <!-- <b-row class="m-2">
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
  </b-row> -->
  <b-row class="m-3">
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
            <b-badge v-else-if="loc.pop < 20" variant="success">unpopular</b-badge>
          </b-list-group-item>
        </b-list-group>
      </b-card>
    </b-col>
    <b-col sm="12" md="6">
      <b-card 
        header="Popular days" 
        align="center"
        header-border-variant="info"
        header-text-variant="info"
        style="height:235px;margin-bottom:10px;"
      >
        <BarChartD3 :idSvg="'cc_freq_day'" :data_source="cc_freq_day" :options="option_day"></BarChartD3>
      </b-card>
      <b-card 
        header="Popular hours" 
        align="center"
        header-border-variant="info"
        header-text-variant="info"
        style="height:260px"
      >
            <b-form-radio-group
        id="btn-radios-2"
        v-model="selectedDayWeek"
        :options="dayWeek"
        :aria-describedby="ariaDescribedby"
        button-variant="outline-info"
        size="sm"
        name="radio-btn-outline"
        buttons
        @change="refresh"
      ></b-form-radio-group>
        <BarChartD3 :idSvg="'cc_freq_hour'" :data_source="cc_freq_hour" :options="option_hour"></BarChartD3>
      </b-card>
    </b-col>
    <b-col sm="12" md="3">
        <b-card no-body 
          header="Credit Card used"
          header-border-variant="info"
          header-text-variant="info"
          align="center"
        >
        <b-list-group>
          <b-list-group-item v-for="num in cc_number" :id="num.key" :key="num.key" class="d-flex justify-content-between align-items-center numList"  @click="changeCCNumber(num.key)">
            **** **** **** {{num.key}}
            <b-badge variant="info">{{num.value}}</b-badge>
          </b-list-group-item>
        </b-list-group>
      </b-card>
    </b-col>
  </b-row>
</b-container>
</template>


<script>
  import BarChartD3 from "@/components/BarChartD3";
  // import StackedBarChartD3 from "@/components/StackedBarChartD3";
  import $ from 'jquery';
  import {nest} from 'd3-collection';

  const d3 = require('d3');

export default {
  name: 'HomePage',
  components:{
    BarChartD3,
    // StackedBarChartD3
  },
  data: function(){
    return{
      cc_all:[],
      cc_number: [],
      selectedNum: 0,
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
      cc_freq_day:[],
      cc_freq_hour:[],
      option_day: {
          x: d => d.key,
          y: d => d.value,
          width: 600,
          height: 160,
          color: '#11CFB4',
          yDomain: [0,40]
      },
      option_hour: {
          x: d => d.key,
          y: d => d.value,
          width: 600,
          height: 160,
          color: '#1183CF',
          //yDomain: this.selectedDayWeek == 0 ? [0, 100] : [0,50]
      },
    }
  },
  async mounted () {
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

    this.cc_all = await d3.csv("/data/credit-cards-data.csv")
    .then((rows) => {
      var cc = []
      for(var i = 0; i < rows.length; i++){
          var d = {
            location_id: rows[i].location_id,
            weekday_id: +rows[i].weekday_id,
            weekday: rows[i].weekday,
            hour: +rows[i].hour,
            day: rows[i].day,
            number: rows[i].last4ccnum
          }
          cc.push(d);
      }
      return cc;
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
    changeCCNumber: function(numID){
      $('.numList').removeClass('active');  
      $('#' + numID).addClass('active');

      this.selectedNum = numID;
      this.refresh();
    },
    filterFreqDay: function(){
      var location = this.selectedLoc;
      var data = this.cc_all;

      var data_filtered = data.filter( function(d){
        if (location == 0 || d["location_id"] == location) { return d; } 
      });

      var data_grouped = nest()
        .key(function(d) { return d.weekday; })
        .rollup(function(v) { return v.length; })
        .entries(data_filtered);
    
      var fulldata = [
        {key: 'MON', value:0},
        {key: 'TUE', value:0},
        {key: 'WED', value:0},
        {key: 'THU', value:0},
        {key: 'FRI', value:0},
        {key: 'SAT', value:0},
        {key: 'SUN', value:0},
      ]
      
      data_grouped.forEach(function(obj){
        var objIndex = fulldata.findIndex((o => o.key == obj.key));
        fulldata[objIndex].value = obj.value;
      });

      this.cc_freq_day = fulldata;
    },
    filterFreqHour: function(){
      var location = this.selectedLoc;
      var weekday = this.selectedDayWeek;
      var data = this.cc_all;

      var data_filtered = data.filter( function(d){
        return (location == 0 || d.location_id == location) && (weekday == 0 || d["weekday_id"] == weekday); 
      });

      var data_grouped = nest()
        .key(function(d) { return d.hour; })
        .rollup(function(v) { return v.length; })
        .entries(data_filtered);
    
      var fulldata = [
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
      
      data_grouped.forEach(function(obj){
        var objIndex = fulldata.findIndex((o => o.key == obj.key));
        fulldata[objIndex].value = obj.value;
      });

      this.cc_freq_hour = fulldata;
    },
     refreshNum: function(){
      var location = this.selectedLoc;
      var weekday = this.selectedDayWeek;
      var data = this.cc_all;

      var data_filtered = data.filter( function(d){
        return (location == 0 || d.location_id == location) && (weekday == 0 || d["weekday_id"] == weekday); 
      });

      var data_grouped = nest()
        .key(function(d) { return d.number;})
        .rollup(function(v) { return v.length; })
        .entries(data_filtered); 
        
      data_grouped = data_grouped.sort((a, b) => parseFloat(b.value) - parseFloat(a.value));
      this.cc_number = data_grouped;
    },
    refresh: function() {
      this.filterFreqDay();
      this.filterFreqHour();
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