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
        svg.append("g").attr("id",'points_path')
        svg.append("g").attr("id",'price')

        return {
            width,
            height,
            svg,
            projection,
            path
        };
    }

    function addPoints(data, { width = 800, height = 550, scale = 500000} = {}){
        
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
            .attr("fill-opacity", 0.5)
            .attr("stroke", "#fff")
            .attr("stroke-width", 0.5);
    }

    function addPaths(data, { width = 800, height = 550, scale = 500000} = {}){
        
        const projection = d3.geoMercator()
            .center([24.8699, 36.07])
            .scale(scale)
            .translate([width / 2, height / 2]);

        var line = d3.line()
            .x(function(d) { return projection([d.long, d.lat])[0]; }) 
            .y(function(d) { return projection([d.long, d.lat])[1]; }) 

        const gpoints = d3.select('#points')
        const gpoints_path = d3.select('#points_path')

        gpoints
            .selectAll("circle")
            .data(data)
            .join("circle")
            .attr("cx", d =>  projection([d.long, d.lat])[0])
            .attr("cy", d =>  projection([d.long, d.lat])[1])
            .attr("r",  2)
            .style("fill",  d =>  d.color)
            .style("stroke", d =>  d.color)
            .attr("fill-opacity", 0.5)
            .attr("stroke", "#fff")
            .attr("stroke-width", 0.5);

        gpoints_path
            .append("path")
            .datum(data) 
            .attr("class", "line") 
            .attr("d", line)
            .style("fill", "none")
            .style("stroke",  d =>  d.color)
            .style("stroke-width", "5");
    }

    function addPrice(data, { width = 800, height = 550, scale = 500000} = {}){
        
        const projection = d3.geoMercator()
            .center([24.8699, 36.07])
            .scale(scale)
            .translate([width / 2, height / 2]);

        const gpoints = d3.select('#price')

        gpoints
            .selectAll("circle")
            .data(data)
            .join("circle")
            .attr("cx", d =>  projection([d.long, d.lat])[0])
            .attr("cy", d =>  projection([d.long, d.lat])[1])
            .attr("r",  4)
            .style("fill",  d =>  d.color)
            .style("stroke", d =>  d.color)
            .attr("fill-opacity", 0.5)
            .attr("stroke", "#fff")
            .attr("stroke-width", 0.5);
    }

    export default {
        name: "MapD3",
        props:{
            datetime: String,
            employees: Object,
            colors: Object,
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
            this.gps = await d3.csv("/data/gps_all_data.csv")
            .then((rows) => {
                var gs = []
                for(var i = 0; i < rows.length; i++){
                    var g = {
                        timestamp: rows[i].timestamp,
                        id: +rows[i].id_employee,
                        lat: rows[i].lat,
                        long: +rows[i].long,
                        color: this.colors[+rows[i].id_employee],
                        location: rows[i].location,
                        price: rows[i].price
                    }
                    gs.push(g);
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
                if (this.allDay)
                    addPaths(this.data_points);
                else {
                    addPoints(this.data_points);
                    addPrice(this.data_points);
                }
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
