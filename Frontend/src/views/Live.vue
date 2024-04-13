<template>
    <VContainer>
        <VResponsive>
            <VRow>
                <VCol cols="3">
                    <VCard>
                        <figure class="highcharts-figure"> 
                        <div id="container_1"></div>
                        </figure>
                    </VCard>
                    </VCol>
                    <VCol cols="6">
                    <VCard>
                        <h3 align="center">Temperarture</h3>
                        <VBtn align="center" variant="tonal" rounded="lg" text="Convert" @click="Convertor();"></VBtn>
                    </VCard>
                    </VCol>
                    <VCol cols="3">
                    <VCard>
                        <figure class="highcharts-figure"> 
                        <div id="container_2"></div>
                        </figure>
                    </VCard>
                </VCol>
            </VRow>
            <VRow>
                <VCol cols="12">
                    <VSheet>
                        <figure class="highcharts-figure">
                        <div id="container_3"></div>
                        </figure>
                    </VSheet>
                </VCol>
            </VRow>
            <VRow>
                <VCol cols="12">
                    <figure class="highcharts-figure">
                        <div id="container_4"></div>
                    </figure>
                </VCol>
            </VRow>
        </VResponsive>
    </VContainer>
    </template>
    
    <script setup>
    // IMPORTS
    import { ref,reactive,watch ,onMounted,onBeforeUnmount,computed } from "vue";  
    import { useRoute ,useRouter } from "vue-router";
    import { useMqttStore} from "@/store/mqttStore";
    import {storeToRefs} from "pinia";
    import {useAppStore} from "@/store/appStore";
    import Highcharts from "highcharts";
    import more from "highcharts/highcharts-more";
    import exporting from "highcharts/modules/exporting";
    more(Highcharts);
    exporting(Highcharts);
    
    // VARIABLES
    const router      = useRouter();
    const route       = useRoute(); 
    const Mqtt        = useMqttStore();
    const appStore    = useAppStore();
    const {payload, payloadTopic} = storeToRefs(Mqtt);
    const Soil_Moisture_Gauge = ref(null);
    const Gauge = ref(null);
    const Humid_ring = ref(null);
    const HEAT_INDEX_ring = ref(null);
    const points = ref(600);
    const shift = ref(false);
   
    const  pressure = computed(()=>{
        if(!!payload.value){
      return '${payload.value.pressure.toFixed(2)}';
    }});

    const humidity= computed(()=>{
    if(!!payload.value){
      return '${payload.value.humidity.toFixed(2)}';
    }});

    // const percent= computed(()=>{
    // if(!!payload.value){
    //   return '${payload.value.percent.toFixed(2)}';
    // }});



    
    // WATCHERS
    watch(payload,(data)=> { 

        if(points.value>0){
            points.value--;
        }
        else {
            shift.value = true;
        }

        Soil_Moisture_Gauge.value.series[0].points[0].update(0);
        Gauge.value.series[0].points[0].update(1000);
        Humid_ring.value.series[0].points[0].update(1000);
        HEAT_INDEX_ring.value.series[0].points[0].update(1000);

      console.log(data.percent);
      if (data.percent > 98 || data.percent < 2) {
          isActive.value = true;
        } else {
          isActive.value = false;
        } 
    });

    
    // FUNCTIONS
    onMounted(()=>{
      // THIS FUNCTION IS CALLED AFTER THIS COMPONENT HAS BEEN MOUNTED
      CreateChart();
      Mqtt.connect();

      setTimeout(()=>{
        Mqtt.subscribe("620156117");
        Mqtt.subscribe("620156117_sub");
      },3000);
    });



    const CreateChart= async()=>{
        Soil_Moisture_Gauge.value=Highcharts.chart("container_3",{
            title: { text: 'Soil Moisture', align: 'left'},
            // the value axis
            yAxis: {
            min: 0,
            max: 100,
            tickPixelInterval: 72,
            tickPosition: 'inside',
            tickColor: Highcharts.defaultOptions.chart.backgroundColor || '#FFFFFF',
            tickLength: 20,
            tickWidth: 2,
            minorTickInterval: null,
            labels: { distance: 20, style: { fontSize: '14px' } },
            lineWidth: 0,
            plotBands: [{ from: 0, to: 25, color: '#DF5353', thickness: 20 }, { from: 25, to: 60, color: '#DDDF0D', thickness: 20
            }, { from: 60, to: 100, color: '#55BF3B', thickness: 20 }]
            },
            tooltip: { shared:true, },
            pane: { startAngle: -90, endAngle: 89.9, background: null, center: ['50%', '75%'], size: '110%' },
            series: [{
            type:'gauge',
            name: 'Soil Moisture',
            data:[0],
            tooltip: { valueSuffix: '%' },
            dataLabels: { format: '{y} %', borderWidth: 0, color: ( Highcharts.defaultOptions.title &&
            Highcharts.defaultOptions.title.style && Highcharts.defaultOptions.title.style.color ) || '#333333', style: { fontSize: '16px' } 
            },
            dial: { radius: '80%', backgroundColor: 'gray', baseWidth: 12, baseLength: '0%', rearLength: '0%' },
            pivot: { backgroundColor: 'gray', radius: 6 }
            }]
        });

        Gauge.value=Highcharts.chart("container_4",{
            title: { text: 'Pressure', align: 'left'},
            // the value axis
            yAxis: {
            min: 0,
            max: 2000,
            tickPixelInterval: 72,
            tickPosition: 'inside',
            tickColor: Highcharts.defaultOptions.chart.backgroundColor || '#FFFFFF',
            tickLength: 20,
            tickWidth: 2,
            minorTickInterval: null,
            labels: { distance: 20, style: { fontSize: '14px' } },
            lineWidth: 0,
            plotBands: [{ from: 0, to: 25000, color: '#DF5353', thickness: 20 }, { from: 25000, to: 60000, color: '#DDDF0D', thickness: 20
            }, { from: 60000, to: 10000, color: '#55BF3B', thickness: 20 }]
            },
            tooltip: { shared:true, },
            pane: { startAngle: -90, endAngle: 89.9, background: null, center: ['50%', '75%'], size: '110%' },
            series: [{
            type:'gauge',
            name: 'Pressure',
            data:[0],
            tooltip: { valueSuffix: ' Pa' },
            dataLabels: { format: '{y} Pa', borderWidth: 0, color: ( Highcharts.defaultOptions.title &&
            Highcharts.defaultOptions.title.style && Highcharts.defaultOptions.title.style.color ) || '#333333', style: { fontSize: '16px' } 
            },
            dial: { radius: '80%', backgroundColor: 'gray', baseWidth: 12, baseLength: '0%', rearLength: '0%' },
            pivot: { backgroundColor: 'gray', radius: 6 }
            }]
        });

        Humid_ring.value = Highcharts.chart("container_1", {
    title: {
        text: "Humidity"
    },
    chart: {
        type: 'pie',
        plotShadow: false,
        height: '250px',
        backgroundColor: 'transparent'
    },
    plotOptions: {
        pie: {
            startAngle: -90,
            endAngle: 90,
            center: ['50%', '75%'],
            size: '110%',
            color:'magenta',
            dataLabels: {
                enabled: false
            }
        }
    },
    tooltip: {
        valueSuffix: '%'
    },
    series: [{
        type: 'pie',
        name: 'Humidity',
        innerSize: '50%',
        data: [0],
        color: 'green'
    }]
});

HEAT_INDEX_ring.value = Highcharts.chart("container_2", {
    title: {
        text: "Heat Index"
    },
    chart: {
        type: 'pie',
        plotShadow: false,
        height: '250px',
        backgroundColor: 'transparent'
    },
    plotOptions: {
        pie: {
            startAngle: -90,
            endAngle: 90,
            center: ['50%', '75%'],
            size: '110%',
            color:'green',
            dataLabels: {
                enabled: false
            }
        }
    },
    tooltip: {
        valueSuffix: '%'
    },
    series: [{
        type: 'pie',
        name: 'Heat Index',
        innerSize: '50%',
        data: [0]
    }]
});
       
    }
//const Convertor(async () => )

    

    
    onBeforeUnmount(()=>{
      // THIS FUNCTION IS CALLED RIGHT BEFORE THIS COMPONENT IS UNMOUNTED
      Mqtt.unsubcribeAll();
    });
    
    </script>
    
    <style scoped>
    /** CSS STYLE HERE */
    
    </style>