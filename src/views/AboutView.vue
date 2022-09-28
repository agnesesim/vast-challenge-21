<template>
<b-container class="bv-example-row">

  <b-row class="m-3">
    <b-col sm="12">
      <b-card 
        no-body
        align="center"
      >
      <b-card-body class="p-0">
        <b-row class="m-0 p-0">
          <b-col md="2" sm="12">       
            <b-form-group
              id="fieldset-1"
              label="Select day"
              label-for="input-1"
              class="m-0 mt-2"
              >
              <b-form-select id="input-1" v-model="selectedDay" :options="days" @change="changeDate"></b-form-select>
            </b-form-group>
          </b-col>
          <b-col md="8" sm="12">
            <b-form-group
              id="fieldset-2"
              class="m-0 mt-2"
              >
              <b-row class="m-0 p-0">
              <b-col sm="2">
                <p>{{time_label[minTimeID]}}</p>
              </b-col>
              <b-col sm="8">
                <b-form-input id="range-1" v-model="selectedTimeID" type="range" :min="minTimeID" :max="maxTimeID"></b-form-input>
                <p><b>{{time_label[selectedTimeID]}}</b></p>
              </b-col>
              <b-col sm="2">
                <p>{{time_label[maxTimeID]}}</p>
              </b-col>
              </b-row>
            </b-form-group>
          </b-col>
          <b-col md="2" sm="12">      
            <br> 
            <p @click="startTime">Play</p>
            <!-- <b-form-checkbox
              id="checkbox-1"
              v-model="allDay"
              name="checkbox-1"
              value="true"
              unchecked-value="false"
            >
              All day
            </b-form-checkbox> -->
          </b-col>
        </b-row>
      </b-card-body>
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
        <MapD3 :datetime="selectedDay + ' ' + time_label[selectedTimeID]" :employees="employees_dict" :colors="employees_colors" :allDay="allDay=='true'"></MapD3>
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
          <b-list-group-item v-for="emp in employees_list" :id="'emp_' + emp.value" :key="emp.value" class="d-flex justify-content-between align-items-center empList" @click="filterEmployees(emp.value)">
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
  import $ from 'jquery';

  const d3 = require('d3');

export default {
  name: 'AboutView',
  components:{
    MapD3
  },
  data: function(){
    return{
      allDay:false,
      days:[],
      selectedDay: '2014-01-06',
      time_all: [],
      time_label: {},
      selectedTimeID: 1,
      minTimeID:1,
      maxTimeID:6407,
      employees_list: [],
      employees_dict: {},
      employees_colors: {},
      interval: 0
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

        var date = new Date(rows[i].timestamp)
        this.time_label[rows[i].id] = date.toLocaleTimeString('it-IT')
      }
      return tt;
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
          this.employees_dict[+e.value] = false;
          this.employees_colors[+e.value] = this.randomColor();
      }
      this.employees_list = emp;
    });

    this.changeDate();
  },
  methods: {
    filterEmployees: function(empID){
      if (empID == null){
        $('.empList').removeClass('active');  
      }
      else {
        this.employees_dict[empID] = !this.employees_dict[empID]
        if (this.employees_dict[empID]){
          $('#emp_'  + empID).addClass('active');
          $('#emp_'  + empID).css("background-color", this.employees_colors[+empID])
        }
        else {
          $('#emp_'  + empID).removeClass('active');
          $('#emp_'  + empID).css("background-color", "#fff")
        }
      }
    },
    changeDate: function(){
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
    incrementTime: function(){
      if (this.selectedTimeID < this.maxTimeID)
        this.selectedTimeID ++;
      else 
        clearInterval(this.interval);
    },
    startTime: function(){
      this.interval = setInterval(this.incrementTime, 100);
    },
    randomColor: function(){
      return '#'+(Math.random()*0xFFFFFF<<0).toString(16);
    }
  }
}
</script>

<style scoped>
.list-group{
    max-height: 600px;
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