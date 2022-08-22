<template>
<b-container class="bv-example-row">
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
        <BarChartD3 :idSvg="'freq_day'" :data_source="freq_day" :options="option_day"></BarChartD3>
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
        button-variant="outline-info"
        size="sm"
        name="radio-btn-outline"
        buttons
        @change="refresh"
      ></b-form-radio-group>
        <BarChartD3 :idSvg="'freq_hour'" :data_source="freq_hour" :options="option_hour"></BarChartD3>
      </b-card>
    </b-col>
    <b-col sm="12" md="3">
        <b-card no-body 
          align="center"
        >
          <b-tabs content-class="m-0 p-0" 
            card
            header-border-variant="info"
            header-text-variant="info"
          >
            <b-tab title="Credit Cards" class="m-0 p-0" active>
              <b-list-group>
                <b-list-group-item v-for="num in cc_number" :id="num.key" :key="num.key" class="d-flex justify-content-between align-items-center numList"  @click="changeCCNumber(num.key)">
                  **** **** **** {{num.key}}
                  <b-badge variant="info">{{num.value}}</b-badge>
                </b-list-group-item>
              </b-list-group>
            </b-tab>
            <b-tab title="Loyalty Cards"  class="m-0 p-0" >
              <b-list-group>
                <b-list-group-item v-for="num in loy_number" :id="num.key" :key="num.key" class="d-flex justify-content-between align-items-center numList"  @click="changeCCNumber(num.key)">
                  {{num.key}}
                  <b-badge variant="info">{{num.value}}</b-badge>
                </b-list-group-item>
              </b-list-group>
            </b-tab>
          </b-tabs>
      </b-card>
    </b-col>
  </b-row>
 
  <b-row class="m-3">
    <b-col sm="12">
      <b-card 
        header="Credit Card paths" 
        align="center"
        header-border-variant="info"
        header-text-variant="info"
      >
      <b-row>
        <b-col md="3" sm="12">
          <b-form-group
            id="fieldset-1"
            label="Select a day"
            label-for="input-1"
            >
            <b-form-select id="input-1" v-model="selectedDay" :options="days" @change="filterTimelineCC();filterTimelineLOY()"></b-form-select>
          </b-form-group>
        </b-col>
        <b-col md="6" sm="12">
          <b-form-group
            id="fieldset-2"
            label="Select one or more credit cards"
            label-for="input-2"
            >
              <multiselect v-model="selectedCC" :options="ccs" :multiple="true" :close-on-select="false" :clear-on-select="false" :preserve-search="true" label="value" track-by="value" :preselect-first="true" @close="filterTimelineCC()">
              </multiselect>
          </b-form-group>
        </b-col>
        <b-col md="3" sm="12">
          <b-form-group
            id="fieldset-3"
            label="Select a loyalty card"
            label-for="input-3"
            >
            <b-form-select id="input-3" v-model="selectedLoy" :options="loys" @change="filterTimelineLOY()"></b-form-select>
          </b-form-group>
        </b-col>
      </b-row>
        <TimeLineD3 :data_source1="datasetCC" :data_source2="datasetLoy" :day="selectedDay" :idSvg="'timeline'"></TimeLineD3>
      </b-card>
    </b-col>
  </b-row>
</b-container>
</template>

<style src="vue-multiselect/dist/vue-multiselect.min.css"></style>
<script>
  import BarChartD3 from "@/components/BarChartD3";
  import TimeLineD3 from "@/components/TimeLineD3";
  import Multiselect from 'vue-multiselect';
  
  import $ from 'jquery';
  import {nest} from 'd3-collection';

  const d3 = require('d3');

export default {
  name: 'HomeView',
  components:{
    BarChartD3,
    TimeLineD3,
    Multiselect
  },
  data: function(){
    return{
      cc_all:[],
      cc_number: [],
      loy_all:[],
      loy_number: [],
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
      freq_day:[],
      freq_hour:[],
      option_day: {
          x: d => d.key,
          y: d => d.value,
          width: 400,
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
      // newww
      selectedDay: '2014-01-06',
      days:[],
      selectedCC: [],
      ccs:[],
      selectedLoy: '',
      loys:[],
      timelineColors: ['#e6194B', '#4363d8', '#ffe119', '#3cb44b', '#f58231', '#f032e6', '#bfef45', '#911eb4', '#42d4f4'],
      datasetCC: [],
      datasetLoy: []
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

    d3.csv("/data/credit-card-list.csv")
    .then((rows) => {
      var ds = []
      for(var i = 0; i < rows.length; i++){
          var d = {
            value: rows[i].number,
            text: rows[i].number,
          }
          ds.push(d);
      }
      this.ccs = ds;
    });

    d3.csv("/data/loyalty-card-list.csv")
    .then((rows) => {
      var ds = []
      for(var i = 0; i < rows.length; i++){
          var d = {
            value: rows[i].number,
            text: rows[i].number,
          }
          ds.push(d);
      }
      this.loys = ds;
    });


    this.cc_all = await d3.csv("/data/credit-card-data.csv")
    .then((rows) => {
      var cc = []
      for(var i = 0; i < rows.length; i++){
          var d = {
            timestamp: rows[i].timestamp,
            location: rows[i].location,
            price: rows[i].price,
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

    this.loy_all = await d3.csv("/data/loyalty-card-data.csv")
    .then((rows) => {
      var ll = []
      for(var i = 0; i < rows.length; i++){
          var d = {
            location: rows[i].location,
            price: rows[i].price,
            location_id: rows[i].location_id,
            weekday_id: +rows[i].weekday_id,
            weekday: rows[i].weekday,
            hour: +rows[i].hour,
            day: rows[i].day,
            number: rows[i].loyaltynum
          }
          ll.push(d);
      }
      return ll;
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
    filterTimelineCC: function(){
      var data = this.cc_all;
      var day = this.selectedDay;
      var colors = this.timelineColors;
      var i = 0;

      var final = []
      this.selectedCC.forEach(element => {
         var data_filtered = data
         .filter( function(d){
            if ( d["day"] == day) { return d; } 
          })
          .filter(function(d){
            if (d["number"] == element.value) { return d; } 
          });

          var points = [];
          data_filtered.forEach(p => {
            points.push([new Date(p.timestamp), p.location, p.price]);
          })

          var creditcard = {
            data: points,
            color: colors[i],
            title: element.value
          }

          i++;
          final.push(creditcard);
      });
      this.datasetCC= final;
    },
    filterTimelineLOY: function(){
      var data = this.loy_all;
      var day = this.selectedDay;
      var num = this.selectedLoy;
      var colors = this.timelineColors;
      var i = 0;

      var final = []
         var data_filtered = data
         .filter( function(d){
            if ( d["day"] == day) { return d; } 
          })
          .filter(function(d){
            if (d["number"] == num) { return d; } 
          });

          var points = [];
          data_filtered.forEach(p => {
            points.push([ p.location,p.location, p.price]);
          })

          var loyaltycard = {
            data: points,
            color: colors[i],
            title: num
          }

          i++;
          final.push(loyaltycard);
  
      console.log(final)
      this.datasetLoy = final;
    },
    filterFreqDay: function(data){
      var location = this.selectedLoc;

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

      return fulldata;
    },
    filterFreqHour: function(data){
      var location = this.selectedLoc;
      var weekday = this.selectedDayWeek;

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

      return fulldata;
    },
    refreshNum: function(data){
      var location = this.selectedLoc;
      var weekday = this.selectedDayWeek;

      var data_filtered = data.filter( function(d){
        return (location == 0 || d.location_id == location) && (weekday == 0 || d["weekday_id"] == weekday); 
      });

      var data_grouped = nest()
        .key(function(d) { return d.number;})
        .rollup(function(v) { return v.length; })
        .entries(data_filtered); 
        
      data_grouped = data_grouped.sort((a, b) => parseFloat(b.value) - parseFloat(a.value));
      return data_grouped;
    },
    refresh: function() {
      this.freq_day = this.filterFreqDay(this.cc_all);
      this.freq_hour = this.filterFreqHour(this.cc_all);
      this.cc_number = this.refreshNum(this.cc_all);
      this.loy_number = this.refreshNum(this.loy_all);
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
  max-width: 1300px;
}
</style>