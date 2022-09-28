<template>
    <b-container class="bv-example-row">
      <b-row class="m-3">
        <b-col md="9" sm="12">
          <b-card 
            header="Daily paths" 
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
            <b-col md="9" sm="12">
              <div class="float-right">
                <b-button variant="danger" @click="selectEmployees()">Clear</b-button>
              </div>
            </b-col>
          </b-row>
            <TimeLineD3 :data_source1="datasetCC" :data_source2="datasetLoy" :day="selectedDay" :idSvg="'timeline'"></TimeLineD3>
          </b-card>
        </b-col>
        <b-col md="3" sm="12">
          <b-card no-body 
            header="Employees"
            header-border-variant="info"
            header-text-variant="info"
            align="center"
          >
            <b-list-group style="max-height:670px">
              <b-list-group-item v-for="emp in employees" :id="'emp_' + emp.id" :key="emp.id" class="d-flex justify-content-between align-items-center empList" @click="selectEmployees(emp.id, emp.value)">
                {{emp.text}}
              </b-list-group-item>
            </b-list-group>
          </b-card>
        </b-col>
      </b-row>
    </b-container>
    </template>
    
    <style src="vue-multiselect/dist/vue-multiselect.min.css"></style>
    <script>
      import TimeLineD3 from "@/components/GraphD3/TimeLineD3";
      
      import $ from 'jquery';
    
      const d3 = require('d3');
    
    export default {
      name: 'HomeView',
      components:{
        TimeLineD3,
      },
      data: function(){
        return{
          days:[],
          employees:[],
          allCC:[],
          allLOY:[],
          selectedDay: '2014-01-06',
          selectedEmp: {},
          selectedCC: [],
          selectedLoy: [],
          timelineColors: ['#e6194B', '#4363d8', '#ffe119', '#3cb44b', '#f58231', '#f032e6', '#bfef45', '#911eb4', '#42d4f4'],
          datasetCC: [],
          datasetLoy: []
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
    
        d3.csv("/data/card-assignments.csv")
          .then((rows) => {
            var empls = []
            for(var i = 0; i < rows.length; i++){
                var p = {
                  id: rows[i].FirstName.replace(' ','') + rows[i].LastName.replace(' ',''),
                  text: rows[i].FirstName + " " + rows[i].LastName,
                  value: { cc: rows[i].last4ccnum, loy: rows[i].loyaltynum }
                }
                empls.push(p);
    
                this.selectedEmp[p.id] = false;
            }
            this.employees = empls;
          });
    
        this.allCC = await d3.csv("/data/credit-card-data.csv")
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
                number: rows[i].last4ccnum,
                employee: rows[i].FirstName + " " + rows[i].LastName
              }
              cc.push(d);
          }
          return cc;
        });
    
        this.allLOY = await d3.csv("/data/loyalty-card-data.csv")
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
                number: rows[i].loyaltynum,
                employee: rows[i].FirstName + " " + rows[i].LastName
              }
              ll.push(d);
          }
          return ll;
        });
      },
      methods: {
        selectEmployees: function(empID, empValue){
          if (empID == null){
            $('.empList').removeClass('active');  
            this.selectedCC = [];
            this.selectedLoy = [];
          }
          else {
            var cc = this.selectedCC;
            var loy = this.selectedLoy;
    
            this.selectedEmp[empID] = !this.selectedEmp[empID]
            if (this.selectedEmp[empID]){
              $('#emp_'  + empID).addClass('active');
              cc.push({ employee: empID,number: empValue.cc})
              loy.push({ employee: empID,number: empValue.loy})
            }
            else {
              $('#emp_'  + empID).removeClass('active');
              this.selectedCC = this.selectedCC.filter(val => val.number != empValue.cc)
              this.selectedLoy = this.selectedLoy.filter(val => val.number != empValue.loy)
            }
          }

          this.filterTimelineCC();
          this.filterTimelineLOY();
        },
        filterTimelineCC: function(){
          var data = this.allCC;
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
                if (d["number"] == element.number) { return d; } 
              });
    
              var points = [];
              data_filtered.forEach(p => {
                points.push([new Date(p.timestamp), p.location, p.price]);
              })
    
              var creditcard = {
                data: points,
                color: colors[i],
                title: element.employee
              }
    
              i++;
              final.push(creditcard);
          });
          this.datasetCC= final;
        },
        filterTimelineLOY: function(){
          var data = this.allLOY;
          var day = this.selectedDay;
          var colors = this.timelineColors;
          var i = 0;
    
          var final = []
           this.selectedLoy.forEach(element => {
            var data_filtered = data
            .filter( function(d){
              if ( d["day"] == day) { return d; } 
            })
            .filter(function(d){
              if (d["number"] == element.number) { return d; } 
            });
    
            var points = [];
            data_filtered.forEach(p => {
              points.push([ p.location,p.location, p.price]);
            })
    
            var loyaltycard = {
              data: points,
              color: colors[i],
              title: element.employee
            }
    
            i++;
            final.push(loyaltycard);
          });
    
          this.datasetLoy = final;
        }
      }
    }
    </script>
    
    <style scoped>
    .list-group{
        margin-bottom: 10px;
        overflow:scroll;
        -webkit-overflow-scrolling: touch;
        margin: 0px;
        padding:0px !important;
    }
    
    .empList .active{
      background-color: teal !important;
    }
    
    .container{
      max-width: 1300px;
    }
    </style>