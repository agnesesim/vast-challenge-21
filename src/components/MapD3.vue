<template>
    <div>
        <svg id="map" width='100%' height='550'>
        </svg>
    </div>
</template>

<script>
    const d3 = require('d3');

    function getMap(struct, { width = 800, height = 550, scale = 500000} = {}) {
        const svg = d3.select('#map');
        const projection = d3.geoMercator()
            .center([24.8699, 36.07])
            .scale(scale)
            .translate([width / 2, height / 2])
        
        const path = d3.geoPath()
            .projection(projection);
        
        svg.append('g').attr("id",'map')
            .selectAll('path')
            .data(struct.features)
            .enter().append('path')
            .attr('d', path)
            .style('fill', 'grey')
            .style('stroke', '#ccc');
        
        svg.append("g").attr("id",'points')
        return {
            width,
            height,
            svg,
            projection,
            path
        };
    }

    function addGPS(data, { width = 800, height = 550, scale = 500000} = {}){
        
        const projection = d3.geoMercator()
            .center([24.8699, 36.07])
            .scale(scale)
            .translate([width / 2, height / 2]);

        const gpoints = d3.select('#points')
        gpoints
        .selectAll("circle")
        .data(data)
        .join("circle")
        .attr("cx", d =>  projection([d.long, d.lat])[0])
        .attr("cy", d =>  projection([d.long, d.lat])[1])
        .attr("r",  4)
        .style("fill",  d =>  d.color)
        .style("stroke", d =>  d.color)
        .attr("fill", "black")
        .attr("fill-opacity", 0.5)
        .attr("stroke", "#fff")
        .attr("stroke-width", 0.5);



        // .attr("fill", "brown")
        // .attr("fill-opacity", 0.5)
        // .attr("stroke", "#fff")
        // .attr("stroke-width", 0.5)
        // .selectAll("circle")
        // .data(data)
        // .join("circle")
        // .attr("cx", d =>  projection([d.long, d.lat])[0])
        // .attr("cy", d =>  projection([d.long, d.lat])[1])
        // .attr("r",  4)
        // .style("fill", 'rgba(255, 0, 0, 0.3)')
        // .style("stroke", 'rgba(255, 0, 0, 0.3)');
    }

    export default {
        name: "MapD3",
        props:{
            datetime: String,
            employees: Object,
            allDay: Boolean
        },
        data: function(){
            return {
                data_structure: {},
                data_points: [],
                gps: [],
                map: {},
            }
        },
        async mounted(){

            this.data_structure =  await d3.json("/data/Abila1.json")
            this.gps = await d3.csv("/data/gps_all.csv")
            .then((rows) => {
                var gs = []
                for(var i = 0; i < rows.length; i++){
                    var g = {
                        timestamp: rows[i].timestamp,
                        id: +rows[i].id_employee,
                        lat: rows[i].lat,
                        long: +rows[i].long,
                        color: this.employees[+rows[i].id_employee+100]
                    }
                    gs.push(g);
                    i++;
                }
                return gs;
            });

            this.map = getMap(this.data_structure)
            this.refresh();
        },
        methods: {
            filterPoints: function(){
                var time = this.datetime;
                var emps = this.employees;
                var data = this.gps;
                var all_day = this.allDay

                var data_filtered = data.filter( function(d){
                    if (all_day)
                        return emps[d["id"]] == true
                    else
                        return d["timestamp"] == time && emps[d["id"]] == true
                });

                this.data_points = data_filtered;
            },
            refresh: function(){
                this.filterPoints();
                addGPS(this.data_points);
            }
        },
        watch:{
           datetime: function(){
               this.refresh();
           },
           employees: function(){
               this.refresh();
           }
        }
    }
</script>
