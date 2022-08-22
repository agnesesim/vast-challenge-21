<template>
    <div id="external">
        <!-- <svg :id="this.idSvg" width='100%' height='400'>
        </svg> -->
    </div>
</template>

<script>
    const d3 = require('d3');
    import $ from 'jquery';

    function TimeLine(data1,
        data2,
        minDate = new Date("2014-01-06 00:00:00"),
        maxDate = new Date("2014-01-06 23:59:00"),
        {
            marginTop = 40, // the top margin, in pixels
            marginRight = 0, // the right margin, in pixels
            marginBottom = 0, // the bottom margin, in pixels
            marginLeft = 110, // the left margin, in pixels
            width = 950, // the outer width of the chart, in pixels
            height = 500, // the outer height of the chart, in pixels
            yLabel, // a label for the y-axis
            yDomain = ["Ouzeri Elian","Brewed Awakenings","Hallowed Grounds","Roberts and Sons","Shoppers' Delight","Abila Airport","Carlyle Chemical Inc.","Kronos Pipe and Irrigation","Frank's Fuel","Maximum Iron and Steel","Katerina s Café","Ahaggo Museum","Kronos Mart","Nationwide Refinery","Hippokampos","Coffee Shack","Stewart and Sons Fabrication","Albert's Fine Clothing","Frydos Autosupply n' More","Desafio Golf Course","Abila Zacharo","Guy's Gyros","Gelatogalore","General Grocer","Kalami Kafenion","Octavio's Office Supplies","Jack's Magical Beans","Coffee Cameleon","U-Pump","Chostus Hotel","Brew've Been Served","Bean There Done That","Daily Dealz","Abila Scrapyard"]
        } = {},
        ) {
            console.log('data1')
            console.log(data1)
            console.log('data2')
            console.log(data2)
        $('#external').empty();
        // const svg = d3.select('#'+id);
        // width = svg.node().getBoundingClientRect().width;
        
        const div = d3.select('#external');
        width = div.node().getBoundingClientRect().width;

        // Construct scales, axes, and formats.
        const xScale = d3.scaleTime().domain([minDate, maxDate]).range([marginLeft, width - marginRight - width/5]);          
        const yScale = d3.scalePoint().domain(yDomain).range([height - marginBottom, marginTop]);      
        const xAxis = d3.axisBottom().scale(xScale).tickSizeOuter(0);
        const yAxis = d3.axisLeft().scale(yScale).ticks(height / 40);
        
        var svg = d3.select("#external")
            .append("svg")
            .attr("width", width + marginLeft + marginRight)
            .attr("height", height + marginTop + marginBottom)

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

        // -1- Create a tooltip div that is hidden by default:
        var tooltip = d3.select("#external")
            .append("div")
            .style("opacity", 0)
            .attr("class", "tooltip")
            .style("background-color", "grey")
            .style("border-radius", "5px")
            .style("padding", "10px")
            .style("color", "white")
        // -2- Create 3 functions to show / update (when mouse move but stay on same circle) / hide the tooltip
        const showTooltip = function(event, d) {
            console.log(d)
            tooltip
            .transition()
            .duration(200)
            tooltip
            .style("opacity", 1)
            .html(d[1] + ": " + d[2] + " $")
            .style("left", (event.x) + "px")
            .style("top", (event.y) + "px")
        }
        const hideTooltip = function() {
            tooltip
            .transition()
            .duration(200)
            .style("opacity", 0)
        }
        var line = d3.line()
            .x(function(d) { return xScale(d[0]); }) 
            .y(function(d) { return yScale(d[1]); }) 
        
        var i = 0;
        data1.forEach(element => {
             svg.append('g')
                .selectAll("dot")
                .data(element.data)
                .enter()
                .append("circle")
                .attr("cx", function (d) { return xScale(d[0]); } )
                .attr("cy", function (d) { return yScale(d[1]); } )
                .attr("r", 5)
                .style("fill", element.color) 
                .on("mouseover", showTooltip )
                .on("mouseleave", hideTooltip )

            svg.append("path")
                .datum(element.data) 
                .attr("class", "line") 
                .attr("d", line)
                .style("fill", "none")
                .style("stroke", element.color)
                .style("stroke-width", "2");
            
             // Add one dot in the legend for each name.
            svg.selectAll("legend_dot")
                .data(element.data)
                .enter()
                .append("circle")
                .attr("cx", marginLeft + i*60)
                .attr("cy",  10)
                .attr("r", 4)
                .style("fill", element.color)

            svg.selectAll("legend_title")
                .data(element.data)
                .enter()
                .append("text")
                .attr("x", marginLeft + 7 + i*60)
                .attr("y", 12)
                .style("fill", element.color)
                .text(element.title)
                .attr("text-anchor", "left")
                .style("alignment-baseline", "middle")  
            i++;
        });
       
       var j = 0;
       data2.forEach(element => {
        console.log(element)
            svg.append('g')
                .selectAll("dot")
                .data(element.data)
                .enter()
                .append("circle")
                .attr("cx", width - 100 + j*10 )
                .attr("cy", function (d) { return yScale(d[0]); } )
                .attr("r", 5)
                .style("fill", "black") 
                .on("mouseover", showTooltip )
                .on("mouseleave", hideTooltip )

             j++;
       })
        svg.append("g")
            .attr("transform", `translate(0,${height - marginBottom})`)
            .call(xAxis);

        // return svg.node();
    }

    export default {
        name: "TimeLineD3",
        props:{
            data_source1: Array,
            data_source2: Array,
            day: {
                type: String,
                default: () => { return '2014-01-06'}
            },
            options: Object,
            idSvg: {
                type: String,
                default: 'idDiDefault'
            }
        },
        // data: function(){
        //     // return {
        //     //     chart: TimeLine(this.data_source1, this.options, this.idSvg),
        //     // }
        // },
        mounted(){
          this.refreshPlot();
        },
        methods:{
          refreshPlot: function(){
            var min_date = new Date( this.day + " 00:00:00");
            var max_date = new Date( this.day + " 23:59:00");
            // this.chart = TimeLine(this.data_source1, min_date, max_date, this.options, this.idSvg)
            TimeLine(this.data_source1,this.data_source2, min_date, max_date, this.options)
          }
        },
        watch:{
           data_source1: function(){
               this.refreshPlot();
           },
           data_source2: function(){
               this.refreshPlot();
           },
           day: function(){
               this.refreshPlot();
           },
        }
    }
</script>
