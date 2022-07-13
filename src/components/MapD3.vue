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
        .attr("fill", "brown")
        .attr("fill-opacity", 0.5)
        .attr("stroke", "#fff")
        .attr("stroke-width", 0.5)
        .selectAll("circle")
        .data(data)
        .join("circle")
        .attr("cx", d =>  projection([d.long, d.lat])[0])
        .attr("cy", d =>  projection([d.long, d.lat])[1])
        .attr("r",  4)
        .style("fill", 'rgba(255, 0, 0, 0.3)')
        .style("stroke", 'rgba(255, 0, 0, 0.3)');
    }

    export default {
        name: "MapD3",
        props:{
            datetime: String,
        },
        data: function(){
            return {
                data_structure: {},
                data_employees: [],
                gps: [],
                map: {}
            }
        },
        async mounted(){
            this.data_structure =  await d3.json("/data/Abila1.json")
            console.log(this.data_structure)
            this.gps = await d3.csv("/data/gps_clean.csv")
            .then((rows) => {
                var gs = []
                for(var i = 0; i < rows.length; i++){
                    var g = {
                        timestamp: rows[i].Timestamp,
                        id: +rows[i].id,
                        lat: rows[i].lat,
                        long: +rows[i].long,
                    }
                    gs.push(g);
                }
                return gs;
            });

            this.map = getMap(this.data_structure)
            this.refresh();
        },
        methods: {
            filterEmployees: function(){
                var time = this.datetime;
                var data = this.gps;

                var data_filtered = data.filter( function(d){
                    if (d["timestamp"] == time) { return d; } 
                });
                
                this.data_employees = data_filtered;
            },
            refresh: function(){
                this.filterEmployees();
                addGPS(this.data_employees);
            }
        },
        watch:{
           datetime: function(){
               this.refresh();
           }
        }
    }
</script>
