<template>
    <div>
        <h1>Employees</h1>
            <b-table hover 
                :items="employees" 
                :fields="columns"
                :current-page="currentPage"
                :per-page="perPage"
            ></b-table>
            <b-row>
                <b-col sm="6"> 
                </b-col>
                <b-col sm="6"> 
                    <b-pagination
                        v-model="currentPage"
                        :total-rows="totalRows"
                        :per-page="perPage"
                        align="fill"
                        size="sm"
                        class="my-0"
                    ></b-pagination>
                </b-col>
            </b-row>
            
      <!-- <b-list-group>
        <b-list-group-item v-for="p in employees" :key="p.id">{{p.name}}</b-list-group-item>
      </b-list-group> -->
    </div>
</template>

<script>
  const d3 = require('d3');

export default {
  name: 'TableEmployees',
  data: function(){
    return{
      employees:[],
      columns: [
          { key: 'id', sortable: true },
          { key: 'name', sortable: true },
          { key: 'job', sortable: true },
          { key: 'field', sortable: false }
        ],
        totalRows: 1,
        currentPage: 1,
        perPage: 5,
        pageOptions: [5, 10, 15, { value: 100, text: "Show a lot" }],
    }
  },
    mounted() {
      d3.csv("/data/car-assignments.csv")
        .then((rows) => {
          var empls = []
          for(var i = 0; i < rows.length; i++){
              var p = {
                id: rows[i].CarID,
                name: rows[i].FirstName + " " + rows[i].LastName,
                job: rows[i].CurrentEmploymentTitle,
                field: rows[i].CurrentEmploymentType
              }
              empls.push(p);
          }
          this.employees = empls;
        });

        this.totalRows = this.employees.length
  },
  
}
</script>
