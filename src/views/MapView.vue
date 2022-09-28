<template>
    <b-container class="bv-example-row">
    
      <b-row class="m-3">
        <b-col sm="12">
          <b-card 
            align="center"
            style=";margin-bottom:10px;"
          >
            <b-row>
              <b-col md="3" sm="12">       
                <b-form-group
                  id="fieldset-1"
                  label="Select day"
                  label-for="input-1"
                  >
                  <b-form-select id="input-1" v-model="selectedDay" :options="days" @change="refresh"></b-form-select>
                </b-form-group>
              </b-col>
              <b-col md="7" sm="12">
                <b-form-group
                  id="fieldset-2"
                  label="Slide to change time"
                  label-for="range-1"
                  >
                  <b-form-input id="range-1" v-model="selectedTimeID" type="range" :min="minTimeID" :max="maxTimeID"></b-form-input>
                </b-form-group>
              </b-col>
              <b-col md="2" sm="12">
                <div class="mt-2"><b>{{ time[selectedTimeID] }}</b></div>
              </b-col>
            </b-row>
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
            <MapD3 :datetime="time[selectedTimeID]" :allDay="false"></MapD3>
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
      },
      data: function(){
        return{
          employees: [],
          selectedDay: '01-06-2014',
          days:[],
          time_all: [],
          time: {},
          selectedTimeID: 1,
          minTimeID:1,
          maxTimeID:6407,
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
        
        this.time_all = await d3.csv("/data/time.csv")
        .then((rows) => {
          var tt = []
          for(var i = 0; i < rows.length; i++){
            var d = {
              time_id: +rows[i].id,
              timestamp: rows[i].timestamp,
              day: rows[i].day
            }
            tt.push(d);
          }
          return tt;
        });
        
        d3.csv("/data/time.csv")
        .then((rows) => {
          for(var i = 0; i < rows.length; i++){
            this.time[rows[i].id] = rows[i].timestamp
          }
        });
    
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
    
        this.refresh();
      },
      methods: {
        filterTime: function(){
          var day = this.selectedDay;
          var data = this.time_all;
    
          var data_filtered = data.filter( function(d){
            if (day == '01-01-1900' || d["day"] == day) { return d; } 
          });
    
          var ids = data_filtered.map(item => item.time_id)
    
          this.minTimeID = Math.min.apply(Math, ids) 
          this.maxTimeID = Math.max.apply(Math, ids) 
          this.selectedTimeID = this.minTimeID
        },
        refresh: function() {
          this.filterTime()
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