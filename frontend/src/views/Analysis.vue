<template>
  <v-container fluid class="bg-surface">
    <v-row class="mx-auto" style="max-width: 1200px;">
      <v-col cols="12" class="py-1">
        <v-row>
          <v-col>
            <v-sheet class="pa-2" height="250">
              <p>Enter date range for Analysis</p>
              <v-divider></v-divider>
              <v-text-field v-model="start" label="Start date" type="date" density="compact" variant="solo-inverted"
                class="mr-5" style="max-width: 300px;" flat></v-text-field>
              <v-text-field v-model="end" label="End date" type="date" density="compact" variant="solo-inverted"
                style="max-width: 300px;" flat></v-text-field>
              <v-spacer></v-spacer>
              <VBtn @click="updateLineCharts();" text="Analyze" color="primary" variant="tonal"></VBtn>
            </v-sheet>
          </v-col>
          <v-col cols="3" class="d-flex justify-center">
            <v-card title="Temperature" width="250" variant="outlined" color="primary" density="compact" rounded="lg">
              <v-card-item class="mb-n5">
                <v-chip-group class="d-flex flex-row justify-center" color="primaryContainer" variant="flat">
                  <v-tooltip location="start">
                    <template v-slot:activator="{ on, attrs }">
                      <v-chip v-bind="attrs" v-on="on">{{ temperature.min }}</v-chip>
                    </template>
                    Min
                  </v-tooltip>
                  <v-tooltip location="top">
                    <template v-slot:activator="{ on, attrs }">
                      <v-chip v-bind="attrs" v-on="on">{{ temperature.range }}</v-chip>
                    </template>
                    Range
                  </v-tooltip>
                  <v-tooltip location="end">
                    <template v-slot:activator="{ on, attrs }">
                      <v-chip v-bind="attrs" v-on="on">{{ temperature.max }}</v-chip>
                    </template>
                    Max
                  </v-tooltip>
                </v-chip-group>
              </v-card-item>
              <v-card-item align="center">
                <span class="text-h1 text-primary font-weight-bold">{{ temperature.avg }}</span>
              </v-card-item>
            </v-card>
          </v-col>
          <v-col cols="3" class="d-flex justify-center">
            <v-card title="Humidity" width="250" variant="outlined" color="primary" density="compact" rounded="lg">
              <v-card-item class="mb-n5">
                <v-chip-group class="d-flex flex-row justify-center" color="primaryContainer" variant="flat">
                  <v-tooltip location="start">
                    <template v-slot:activator="{ on, attrs }">
                      <v-chip v-bind="attrs" v-on="on">{{ humidity.min }}</v-chip>
                    </template>
                    Min
                  </v-tooltip>
                  <v-tooltip location="top">
                    <template v-slot:activator="{ on, attrs }">
                      <v-chip v-bind="attrs" v-on="on">{{ humidity.range }}</v-chip>
                    </template>
                    Range
                  </v-tooltip>
                  <v-tooltip location="end">
                    <template v-slot:activator="{ on, attrs }">
                      <v-chip v-bind="attrs" v-on="on">{{ humidity.max }}</v-chip>
                    </template>
                    Max
                  </v-tooltip>
                </v-chip-group>
              </v-card-item>
              <v-card-item align="center">
                <span class="text-h1 text-primary font-weight-bold">{{ humidity.avg }}</span>
              </v-card-item>
            </v-card>
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="12" class="d-flex justify-end">
            <VBtn @click="updateCards();" text="Analyze" color="primary" variant="tonal">Analyze</VBtn>
          </v-col>
        </v-row>
      </v-col>
      <v-col cols="12">
        <v-row>
          <v-col cols="12">
            <figure class="highcharts-figure">
              <div id="container"></div>
            </figure>
          </v-col>
          <v-col cols="12">
            <figure class="highcharts-figure">
              <div id="container0"></div>
            </figure>
          </v-col>
        </v-row>
      </v-col>
      <v-col cols="12">
        <v-row>
          <v-col cols="12" border>
            <figure class="highcharts-figure">
              <div id="container1"></div>
            </figure>
          </v-col>
          <v-col cols="12">
            <figure class="highcharts-figure">
              <div id="container2"></div>
            </figure>
          </v-col>
          <v-col cols="12">
            <figure class="highcharts-figure">
              <div id="container3"></div>
            </figure>
          </v-col>
        </v-row>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup>
import { useMqttStore } from '@/store/mqttStore'; // Import Mqtt Store
import { storeToRefs } from "pinia";
// IMPORTS
import { useAppStore } from "@/store/appStore";
import { onMounted, onBeforeUnmount, ref } from 'vue';
// Highcharts, Load the exporting module and Initialize exporting module.
import Highcharts from 'highcharts';
import more from 'highcharts/highcharts-more';
import Exporting from 'highcharts/modules/exporting';
Exporting(Highcharts);
more(Highcharts);

// VARIABLES
const Mqtt = useMqttStore();
const AppStore = useAppStore();
const { payload, payloadTopic } = storeToRefs(Mqtt);
const tempHiChart = ref(null); // Chart object
const humidityChart = ref(null); // Chart object
const freqChart = ref(null);
const scatterChart = ref(null);

onMounted(() => {
  // THIS FUNCTION IS CALLED AFTER THIS COMPONENT HAS BEEN MOUNTED
  CreateCharts();
  Mqtt.connect(); // Connect to Broker located on the backend
  setTimeout(() => {
    // Subscribe to each topic
    Mqtt.subscribe("620155671");
    Mqtt.subscribe("620155671_sub");
  }, 3000);
});

onBeforeUnmount(() => {
  // THIS FUNCTION IS CALLED RIGHT BEFORE THIS COMPONENT IS UNMOUNTED
  // unsubscribe from all topics
  Mqtt.unsubcribeAll();
});
const start = ref("");
const end = ref("");
const temperature = ref({
  min: 0,
  range: 0,
  max: 0,
  avg: 0
});

const humidity = ref({
  min: 0,
  range: 0,
  max: 0,
  avg: 0
});

const CreateCharts = async () => {
  // TEMPERATURE CHART
  tempHiChart.value = Highcharts.chart('container', {
    chart: { zoomType: 'x' },
    title: { text: 'Air Temperature and Heat Index Analysis', align: 'left' },
    yAxis: {
      title: { text: 'Air Temperature & Heat Index', style: { color: '#000000' } },
      labels: { format: '{value} °C' }
    },
    xAxis: {
      type: 'datetime',
      title: { text: 'Time', style: { color: '#000000' } },
    },
    tooltip: { shared: true, },
    series: [
      {
        name: 'Temperature',
        type: 'spline',
        data: [],
        turboThreshold: 0,
        color: Highcharts.getOptions().colors[0]
      },
      {
        name: 'Heat Index',
        type: 'spline',
        data: [],
        turboThreshold: 0,
        color: Highcharts.getOptions().colors[1]
      }],
  });



  // HUMIDITY CHART
  humidityChart.value = Highcharts.chart('container0', {
    chart: { zoomType: 'x' },
    title: { text: 'Humidity Analysis', align: 'left' },
    yAxis: {
      title: { text: 'Humidity', style: { color: '#000000' } },
      labels: { format: '{value} %' }
    },
    xAxis: {
      type: 'datetime',
      title: { text: 'Time', style: { color: '#000000' } }
    },
    tooltip: { shared: true },
    series: [
      {
        name: 'Humidity',
        type: 'spline',
        data: [],
        turboThreshold: 0,
        color: Highcharts.getOptions().colors[2]
      }
    ]
  });


  freqChart.value = Highcharts.chart('container1', {
    chart: {
      type: 'column', zoomType: 'x'
    },
    title: {
      text: 'Corn vs wheat estimated production for 2023'
    },
    subtitle: {
      text:
        'Source: <a target="_blank" ' +
        'href="https://www.indexmundi.com/agriculture/?commodity=corn">indexmundi</a>'
    },
    xAxis: {

      categories: [],
      title: { text: 'Variables' }
    },
    yAxis: {
      min: 0,
      title: {
        text: ''
      }
    },
    tooltip: {
      valueSuffix: ''
    },
    plotOptions: {
      column: {
        pointPadding: 0.2,
        borderWidth: 0
      }
    },
    series: [
      {
        name: 'Temperature',
        turboThreshold: 0,
        data: []
      },
      {
        name: 'Heat index',
        turboThreshold: 0,
        data: []
      },
      {
        name: 'Humidity',
        turboThreshold: 0,
        data: []
      }
    ]
  });



  //       freqChart.value = Highcharts.chart('container2', {
  //   chart: { type: 'column', zoomType: 'x' },
  //   title: { text: 'Frequency Distribution Analysis', align: 'left' },
  //   xAxis: {
  //     categories: ['Temperature', 'Humidity', 'Heat Index'],
  //     title: { text: 'Variables' }
  //   },
  //   yAxis: {
  //     min: 0,
  //     title: { text: 'Frequency' }
  //   },
  //   series: [
  //     {
  //       name: 'Temperature',
  //       data: frequencyData.temperature,
  //       color: Highcharts.getOptions().colors[0]
  //     },
  //     {
  //       name: 'Humidity',
  //       data: frequencyData.humidity,
  //       color: Highcharts.getOptions().colors[2]
  //     },
  //     {
  //       name: 'Heat Index',
  //       data: frequencyData.heatindex,
  //       color: Highcharts.getOptions().colors[1]
  //     }
  //   ]
  // });

  // SCATTER CHART
  scatterChart.value = Highcharts.chart('container2', {
    chart: {
      type: 'scatter',
      zoomType: 'x'
    },
    title: {
      text: 'Temperature & Heat Index Correlation Analysis',
      align: 'left'
    },
    subtitle: {
      text: 'Visualize the relationship between Temperature and Heat Index as well as revealing patterns or trends in the data'
    },
    xAxis: {
      title: {
        text: 'Temperature'
      },
      labels: {
        format: '{value} °C'
      }
    },
    yAxis: {
      title: {
        text: 'Heat Index'
      },
      labels: {
        format: '{value} °C'
      }
    },
    tooltip: {
      pointFormat: 'Temperature: {point.x} °C <br/> Heat Index: {point.y} °C'
    },
    series: [{
      name: 'Analysis',
      data: [],
      color: Highcharts.getOptions().colors[0]
    }]
  });

};

const updateLineCharts = async () => {
  if (!!start.value && !!end.value) {
    // Convert output from Textfield components to 10 digit timestamps
    let startDate = new Date(start.value).getTime() / 1000;
    let endDate = new Date(end.value).getTime() / 1000;
    // Fetch data from backend
    const data = await AppStore.getAllInRange(startDate, endDate);
    const tempFreqData = await AppStore.getFrequencyVariable(startDate, endDate, 'temperature');
    const humidityFreqData = await AppStore.getFrequencyVariable(startDate, endDate, 'humidity');
    const heatFreqData = await AppStore.getFrequencyVariable(startDate, endDate, 'heatindex');

    // Create arrays for each plot
    let temperature = [];
    let heatindex = [];
    let humidity = [];


    let hum = [];
    let tmp = [];
    let heat = [];

    tempFreqData.forEach(row => {
      tmp.push({ "x:": row._id, "y": row.count });
      console.log(row);
    });

    humidityFreqData.forEach(row => {
      hum.push({ "x:": row._id, "y": row.count });
      console.log(row);
    });

    heatFreqData.forEach(row => {
      heat.push({ "x:": row._id, "y": row.count });
      console.log(row);
    });

    // Iterate through data variable and transform object to format recognized by highcharts
    data.forEach(row => {
      temperature.push({ "x": row.timestamp * 1000, "y": parseFloat(row.temperature.toFixed(2)) });
      heatindex.push({ "x": row.timestamp * 1000, "y": parseFloat(row.heatindex.toFixed(2)) });
      humidity.push({ "x": row.timestamp * 1000, "y": parseFloat(row.humidity.toFixed(2)) });

    });
    // Add data to Temperature and Heat Index chart
    tempHiChart.value.series[0].setData(temperature);
    tempHiChart.value.series[1].setData(heatindex);
    humidityChart.value.series[0].setData(humidity);

    freqChart.value.series[0].setData(tmp);
    freqChart.value.xAxis[0].setCategories(tempFreqData.map(row => row._id));
    // freqChart.value.series[1].setData(humidity);
    freqChart.value.series[2].setData(hum);
    // freqChart.value.xAxis[2].setCategories(humidityFreqData.map(row => row._id));
    freqChart.value.series[1].setData(heat);
    // freqChart.value.xAxis[1].setCategories(heatFreqData.map(row => row._id));
  }

  


};

const updateCards = async () => {
    // Retrieve Min, Max, Avg, Spread/Range
    if (!!start.value && !!end.value) {
      // 1. Convert start and end dates collected fron TextFields to 10 digit timestamps
      let startDate = new Date(start.value).getTime() / 1000;
      let endDate = new Date(end.value).getTime() / 1000;
      // 2. Fetch data from backend by calling the API functions
      const temp = await AppStore.getTemperatureMMAR(startDate, endDate);
      const humid = await AppStore.getHumidityMMAR(startDate, endDate);
      console.log(humid,temp );


      temperature.max = temp[0].max.toFixed(1);
      //3. complete for min, avg and range
      temperature.min = temp[0].min.toFixed(1);
      temperature.avg = temp[0].avg.toFixed(1);
      temperature.range = (temp[0].max - temp[0].min).toFixed(1);

      //4. complete max, min, avg and range for the humidity variable
      humidity.max = humid[0].max.toFixed(1);
      humidity.min = humid[0].min.toFixed(1);
      humidity.avg = humid[0].avg.toFixed(1);
      humidity.range = (humid[0].max - humid[0].min).toFixed(1);
    }
  }
</script>

<style scoped>
.highcharts-figure {
  margin: 0;
}

Figure {
  border: 2px solid black;
}
</style>