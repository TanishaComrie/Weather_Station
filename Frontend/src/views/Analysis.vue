<template>
    <VContainer>
        <VResponsive>
            <VRow>
                <VCol cols="3" align="center">
                    <v-text-field label="Start Date" type="Date" v-model="start" density="compact" variant="solo-inverted" max-width="300" flat></v-text-field>
                    <v-text-field label="End Date" type="Date" v-model="end" density="compact" variant="solo-inverted" max-width="300" flat></v-text-field>
                    <VSpacer></VSpacer>
                    <VBtn text="Analyze" @click="update();" variant="tonal" rounded="lg"></VBtn>                        
                </VCol>
                <VCol cols="3" height="250">
                    <VCard title="Temperature" subtitle="for the selected period" width="250" height="200" variant="elevated" rounded="lg" density="compact" align="center">
                        <v-card-item>
                            <span class="text-h1 text-black">
                                {{ temperature.avg }}
                            </span>
                            <small>°C</small>
                        </v-card-item>
                    </VCard>
                    </VCol>
                    <VCol cols="3" height="250">
                    <VCard title="Humidity" subtitle="for the selected period" width="250" height="200" variant="elevated" rounded="lg" density="compact" align="center">
                        <v-card-item margin-botton= "-5">
                            <span class="text-h1 text-black">
                                {{ humidity.avg }}
                            </span>
                            <small>%</small>
                        </v-card-item>
                    </VCard>
                </VCol>
                <VCol cols="3" height="250">
                    <VCard title="Pressure" subtitle="for the selected period" width="250" height="200" variant="elevated" rounded="lg" density="compact" align="center">
                        <v-card-item>
                            <span class="text-h1 text-black">
                                {{ pressure.avg }}
                            </span>
                            <small>Pa</small>
                        </v-card-item>
                    </VCard>
                </VCol>
            </VRow>
            <VRow max-width="1200" justify="start">
                <VCol cols="12">
                <figure class="highcharts-figures">
                    <div id="container"></div> 
                </figure>
                </VCol>
                <VCol cols="12">
                    <figure class="highcharts-figures">
                        <div id="container0"></div> 
                    </figure>
                </VCol>
                <VCol cols="12">
                    <figure class="highcharts-figures">
                        <div id="container1"></div> 
                    </figure>
                </VCol>
                <VCol cols="12">
                    <figure class="highcharts-figures">
                        <div id="container2"></div> 
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
        const router                        = useRouter();
        const route                         = useRoute(); 
        const Mqtt                          = useMqttStore();
        const AppStore                      = useAppStore();
        const {payload, payloadTopic}       = storeToRefs(Mqtt);
        const Temp_Heat_Analysis            = ref(null);
        const Humid_Heat_Analysis           = ref(null);
        const Soil_Moisture_Analysis        = ref(null);
        const Pressure_Altitude_Analysis    = ref(null);
        var start                           = ref(null);
        var end                             = ref(null);
        var temperature                     = reactive ({avg:0});
        var humidity                        = reactive ({avg:0});
        var pressure                        = reactive ({avg:0});


    // COMPUTED PROPERTIES
    
    //WATCHERS
    
    // FUNCTIONS
    onMounted(()=>{
      // THIS FUNCTION IS CALLED AFTER THIS COMPONENT HAS BEEN MOUNTED
      CreateCharts();
      update ();
      Mqtt.connect();


      setTimeout(() => {
      // Subscribe to each topic
      Mqtt.subscribe("620156117");
      Mqtt.subscribe("620156117_sub");
    }, 3000);
    });
     const CreateCharts = async()=> {
        // Temperarture and Heat Analysis
        Temp_Heat_Analysis.value= Highcharts.chart("container",{
            title:{text:"Temperature and Heat Index Analysis", align:"left"},
            tooltip:{shared: true,
                    pointFormat: 'Temperature: {point.x} °C <br/> Heat Index: {point.y} %'},
            chart:{zoomType:"x"},
            yAxis: {
                title:{text: "Temperature"},
                labels:{format: '{value} °C'},
            },
            xAxis: {
                title:{text:"Time "},
                labels:{format: '{value} s'}
            },
            series:[{name:"Temperature",
                        type:"line"}, {name:"Heat Index", type:"line"}]
            });
        // Humidity and Heat Analysis
            Humid_Heat_Analysis.value= Highcharts.chart("container0",{
            title:{text:"Humidity and Heat Index Analysis", align:"left"},
            tooltip:{shared: true,
                    pointFormat: 'Humidity: {point.x} % <br/> Heat Index: {point.y} °C'},
            chart:{zoomType:"x"},
            yAxis: {
                title:{text: "Humidity"},
                labels:{format: '{value} %'}
            },
            xAxis: {
                type: "datetime",
                title:{text:"Time"},
                labels:{format: '{value} s'}
            },
            series:[{name:"Humidity",
                        type:"line"}, {name:"Heat Index", type:"line"}]
            });
            // Soil Moisture Vs Time
            Soil_Moisture_Analysis.value= Highcharts.chart("container1",{
            title:{text:"Soil Moisture Analysis", align:"left"},
            tooltip:{shared: true,
                    pointFormat: 'Soil_Moisture: {point.x} % <br/> Time: {point.y} s'},
            chart:{zoomType:"x"},
            yAxis: {
                title:{text: "Soil Moisture Percentage"},
                labels:{format: '{value} %'},
                min: 0,
                max:100
            },
            xAxis: {
                type: "datetime",
                title:{text:"Time"},
                labels:{format: '{value} s'}
            },
            series:[{name:"Soil Moisture Percent",
                type:"scatter"}]
            });
            // Pressure Vs Altitude
            Pressure_Altitude_Analysis.value= Highcharts.chart("container2",{
            title:{text:"Altitude and Pressure Analysis", align:"left"},
            tooltip:{shared: true,
                    pointFormat: 'Pressure: {point.x} Pa <br/> Altitude: {point.y} m'},
            chart:{zoomType:"x"},
            yAxis: {
                title:{text: "Altitude"},
                labels:{format: '{value} m'},
            },
            xAxis: {
                title:{text:"Pressure"},
                labels:{format: '{value} Pa'}
            },
            series:[{name:"Pressure",type:"line"}]
            });
    }


    const update = async () => {
        // retriving data from appStore
    if (!!start.value && !!end.value) {
        let start_time = new Date(start.value).getTime() / 1000;
        let end_time = new Date(end.value).getTime() / 1000;
        const data = await AppStore.getAllInRange(start_time, end_time);
       //const average = await AppStore.get_avg(start_time, end_time);
        //  const temp = await AppStore.getTemperatureMMAR(start_time, end_time);
        // const humid = await AppStore.getHumidityMMAR(start_time, end_time);
        // const pressure = await AppStore.getPressureMMAR(start_time, end_time);
        // // const soil_Moisture = await AppStore.getSoil_MoistureMMAR(start_time, end_time);
        // const heat_index = await AppStore.get_Heat_INDEXMMAR(start_time, end_time);
        // // const altitude = await AppStore.get_AltitudeMMAR(start_time, end_time);
        let TempC = [];
        let heatINdex =[];
        let HumiDity = [];
        let PaPress = [];
        let Maltitude =[];
        let moisture_Soil = [];


        data.forEach(row=>{
            if(row.timestamp!=null && row.temperature!=null && row.humidity!= null && row.pressure!=null){
                
            
            TempC.push({
                x: row.timestamp*1000,
                y: parseFloat(row.temperature.toFixed(2)),
            });
            HumiDity.push({
                
                y: parseFloat(row.humidity.toFixed(2)),
            });
            PaPress.push({
                x: row.timestamp*1000,
                y: parseFloat(row.pressure.toFixed(2)),
            });
            Maltitude.push({
                x: row.timestamp*1000,
                y: parseFloat(row.altitude.toFixed(2)),
            });
            heatINdex.push({
                x: row.timestamp*1000,
                y: parseFloat(row['heat index'].toFixed(2)),
            });
            moisture_Soil.push({
                x:  row.timestamp*1000,
                y: parseFloat(row["soil moisture"].toFixed(2)),
            });}
        });

        //sets data to each chart         
        Temp_Heat_Analysis.value.series[0].setData(TempC);
        Temp_Heat_Analysis.value.series[1].setData(heatINdex);
        Humid_Heat_Analysis.value.series[0].setData( HumiDity);
        Humid_Heat_Analysis.value.series[1].setData(heatINdex);
        Soil_Moisture_Analysis.value.series[0].setData(moisture_Soil);
        Pressure_Altitude_Analysis.value.series[0].setData(PaPress);
        Pressure_Altitude_Analysis.value.series[0].setData(Maltitude);

        // prints to the console log
        // console.log(temp);
        // console.log(heat_index);
        // console.log(humid);
        // console.log(soil_Moisture);
        // console.log(pressure);
        // console.log(altitude);

        // //complete avg for the temperatue variable
        // temperature.avg = tempC.avg.toFixed(1);
        // //complete avg for the humidity variable
        // humidity.avg = HumiDity[0].avg.toFixed(1);
        // //complete avg for the pressure variable
        // pressure.avg = pressure[0].avg.toFixed(1);
    }
};
  
    
    onBeforeUnmount(()=>{
      // THIS FUNCTION IS CALLED RIGHT BEFORE THIS COMPONENT IS UNMOUNTED
      Mqtt.unsubcribeAll();
    });
    
    </script>
    
    <style scoped>
    /** CSS STYLE HERE */
    
    </style>
    