<template>
    <div>
        <svg id="map" width='100%' height='400'>
            </svg>
    </div>
</template>

<script>
    const d3 = require('d3');

    function getMap(mapdata, { width = 800, height = 600, scale = 500000} = {}) {
        const svg = d3.select('#map');
        const projection = d3.geoMercator()
            .center([24.8699, 36.07])
            .scale(scale)
            .translate([width / 2, height / 2])
        
        const path = d3.geoPath()
            .projection(projection);
        
        svg.append('g').selectAll('path')
            .data(mapdata.features)
            .enter().append('path')
            .attr('d', path)
            .style('fill', 'grey')
            .style('stroke', '#ccc');
        
        return {
            width,
            height,
            svg,
            projection,
            path
        };
    }

    export default {
        name: "MapD3",
        // props:{
        //     data_map: Object,
        // },
        data: function(){
            return {
                data: {},
                map: {}
            }
        },
        async mounted(){
            this.data =  await d3.json("/data/Abila1.json", function() {
                console.log('ehila')
            })
            this.map = getMap(this.data,  this.idSvg)
        },
        // watch:{
        //    data_map: function(){
        //        this.refreshPlot();
        //    }
        // }
    }
</script>

