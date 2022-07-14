<template>
    <TimeLineD3 :data_source="dataset" :idSvg="'timeline'"></TimeLineD3>
</template>


<script>
  import TimeLineD3 from "@/components/TimeLineD3";

    export default {
    name: 'TimelineView',
    components:{
        TimeLineD3
    },
    data: function(){
        return{
            dataset: 
            [   {   data: [
                    [new Date("2014-01-06 08:07:00"),"Abila Airport"], 
                    [new Date("2014-01-06 11:20:00"),"Maximum Iron and Steel"], 
                    [new Date("2014-01-06 13:50:00"),"U-Pump"], 
                    [new Date("2014-01-06 19:20:00"),"General Grocer"]
                    ],
                    color: '#CC0000'
                },
                {   data: [
                    [new Date("2014-01-06 05:40:00"),"General Grocer"], 
                    [new Date("2014-01-06 10:29:00"),"Coffee Shack"], 
                    [new Date("2014-01-06 13:40:00"),"Kronos Mart"], 
                    [new Date("2014-01-06 19:10:00"),"Abila Airport"]
                    ],
                    color: '#39db24'
                }
            ]
        }
    }
    }
</script>
