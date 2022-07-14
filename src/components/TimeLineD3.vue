<template>
    <div>
        <svg :id="this.idSvg" width='100%' height='400'>
            </svg>
    </div>
</template>

<script>
    const d3 = require('d3');
    import $ from 'jquery';

    function TimeLine(data, {
        marginTop = 20, // the top margin, in pixels
        marginRight = 0, // the right margin, in pixels
        marginBottom = 30, // the bottom margin, in pixels
        marginLeft = 110, // the left margin, in pixels
        width = 950, // the outer width of the chart, in pixels
        height = 400, // the outer height of the chart, in pixels
        yLabel, // a label for the y-axis
        minDate = new Date("2014-01-06 01:00:00"),
        maxDate = new Date("2014-01-06 23:59:00"),
        yDomain = ["Ouzeri Elian","Brewed Awakenings","Hallowed Grounds","Roberts and Sons","Shoppers' Delight","Abila Airport","Carlyle Chemical Inc.","Kronos Pipe and Irrigation","Frank's Fuel","Maximum Iron and Steel","Katerina s Café","Ahaggo Museum","Kronos Mart","Nationwide Refinery","Hippokampos","Coffee Shack","Stewart and Sons Fabrication","Albert's Fine Clothing","Frydos Autosupply n' More","Desafio Golf Course","Abila Zacharo","Guy's Gyros","Gelatogalore","General Grocer","Kalami Kafenion","Octavio's Office Supplies","Jack's Magical Beans","Coffee Cameleon","U-Pump","Chostus Hotel","Brew've Been Served","Bean There Done That","Daily Dealz","Abila Scrapyard"]
        } = {},
        id
        ) {

        $('#'+id).empty();
        const svg = d3.select('#'+id);
        width = svg.node().getBoundingClientRect().width;
        
        // Construct scales, axes, and formats.
        const xScale = d3.scaleTime().domain([minDate, maxDate]).range([marginLeft, width - marginRight]);          
        const yScale = d3.scalePoint().domain(yDomain).range([height - marginBottom, marginTop]);      
        const xAxis = d3.axisBottom().scale(xScale).tickSizeOuter(0);
        const yAxis = d3.axisLeft().scale(yScale).ticks(height / 40);


        svg.append("g")
            .attr("transform", `translate(${marginLeft},0)`)
            .call(yAxis)
            .call(g => g.select(".domain").remove())
            .call(g => g.selectAll(".tick line").clone()
                .attr("x2", width - marginLeft - marginRight)
                .attr("stroke-opacity", 0.1))
            .call(g => g.append("text")
                .attr("x", -marginLeft)
                .attr("y", 10)
                .attr("fill", "currentColor")
                .attr("text-anchor", "start")
                .text(yLabel));

        var line = d3.line()
            .x(function(d) { return xScale(d[0]); }) 
            .y(function(d) { return yScale(d[1]); }) 

        data.forEach(element => {
             svg.append('g')
                .selectAll("dot")
                .data(element.data)
                .enter()
                .append("circle")
                .attr("cx", function (d) { return xScale(d[0]); } )
                .attr("cy", function (d) { return yScale(d[1]); } )
                .attr("r", 4)
                .style("fill", element.color);
    
            svg.append("path")
                .datum(element.data) 
                .attr("class", "line") 
                .attr("d", line)
                .style("fill", "none")
                .style("stroke", element.color)
                .style("stroke-width", "2");

        });
       
        svg.append("g")
            .attr("transform", `translate(0,${height - marginBottom})`)
            .call(xAxis);

        return svg.node();
    }

    export default {
        name: "TimeLineD3",
        props:{
            data_source: Array,
            options: Object,
            idSvg: {
                type: String,
                default: 'idDiDefault'
            }
        },
        data: function(){
            return {
                chart: TimeLine(this.data_source, this.options, this.idSvg),
            }
        },
        mounted(){
          this.refreshPlot();
        },
        methods:{
          refreshPlot: function(){
            this.chart = TimeLine(this.data_source, this.options, this.idSvg)
          }
        },
        watch:{
           data_source: function(){
               this.refreshPlot();
           }
        }
    }
</script>
