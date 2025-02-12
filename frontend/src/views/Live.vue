<template>
  <v-container fluid class="surface" align="center">
    <v-row class="mx-auto" style="max-width: 1200px;">
      <v-col cols="9">
        <figure class="highcharts-figure">
          <div id="container"></div>
        </figure>
      </v-col>
      <v-col cols="3">
        <v-card class="mb-5" max-width="500" color="primaryContainer">
          <v-card-subtitle>Temperature</v-card-subtitle>
          <v-card-item>
            <span class="text-h3 text-onPrimaryContainer">{{ temperature }}</span>
          </v-card-item>
        </v-card>
        <v-card class="mb-5" max-width="500" color="tertiaryContainer">
          <v-card-subtitle>Heat Index (Feels like)</v-card-subtitle>
          <v-card-item>
            <span class="text-h3 text-onTertiaryContainer">{{ heatindex }}</span>
          </v-card-item>
        </v-card>
        <v-card class="mb-5" max-width="500" color="secondaryContainer">
          <v-card-subtitle>Humidity</v-card-subtitle>
          <v-card-item>
            <span class="text-h3 text-onSecondaryContainer">{{ humidity }}</span>
          </v-card-item>
        </v-card>
      </v-col>
    </v-row>
    <v-row class="mx-auto" style="max-width: 1200px;" justify="start">
      <v-col cols="9">
        <figure class="highcharts-figure">
          <div id="container1"></div>
        </figure>
      </v-col>
      <v-col cols="3"></v-col>
    </v-row>
  </v-container>
</template>

<script>
import { ref, onMounted, onBeforeUnmount, watch, computed } from 'vue';
import { useMqttStore } from '@/store/mqttStore'; // Import Mqtt Store
import { storeToRefs } from 'pinia';
import Highcharts from 'highcharts';
import more from 'highcharts/highcharts-more';
import Exporting from 'highcharts/modules/exporting';

Exporting(Highcharts);
more(Highcharts);

export default {
  name: 'Live',
  setup() {
    const Mqtt = useMqttStore();
    const { payload } = storeToRefs(Mqtt);
    const tempHiChart = ref(null); // Temperature and Heat Index chart object
    const humidityChart = ref(null); // Humidity chart object
    const points = ref(10); // Specify the quantity of points to be shown on the live graph simultaneously.
    const shift = ref(false); // Delete a point from the left side and append a new point to the right side of the graph.

    onBeforeUnmount(() => {
      // THIS FUNCTION IS CALLED RIGHT BEFORE THIS COMPONENT IS UNMOUNTED
      // unsubscribe from all topics
      Mqtt.unsubcribeAll();
    });

    onMounted(() => {
      // THIS FUNCTION IS CALLED AFTER THIS COMPONENT HAS BEEN MOUNTED
      CreateCharts();
      Mqtt.connect(); // Connect to Broker located on the backend
      setTimeout(() => {
        // Subscribe to each topic
        Mqtt.subscribe('620155671');
        Mqtt.subscribe('620155671_sub');
      }, 3000);
    });

    watch(payload, (data) => {
      if (points.value > 0) {
        points.value--;
      } else {
        shift.value = true;
      }
      tempHiChart.value.series[0].addPoint(
        { y: parseFloat(data.temperature.toFixed(2)), x: data.timestamp * 1000 },
        true,
        shift.value
      );
      tempHiChart.value.series[1].addPoint(
        { y: parseFloat(data.heatindex.toFixed(2)), x: data.timestamp * 1000 },
        true,
        shift.value
      );
      humidityChart.value.series[0].addPoint(
        { y: parseFloat(data.humidity.toFixed(2)), x: data.timestamp * 1000 },
        true,
        shift.value
      );
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
          title: { text: 'Time', style: { color: '#000000' } }
        },
        tooltip: { shared: true },
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
          }
        ]
      });

      // HUMIDITY CHART
      humidityChart.value = Highcharts.chart('container1', {
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
    };

    // COMPUTED PROPERTIES
    const temperature = computed(() => {
      if (!!payload.value) {
        return `${payload.value.temperature.toFixed(2)} °C`;
      }
      return 'N/A';
    });

    const heatindex = computed(() => {
      if (!!payload.value) {
        return `${payload.value.heatindex.toFixed(2)} °C`;
      }
      return 'N/A';
    });

    const humidity = computed(() => {
      if (!!payload.value) {
        return `${payload.value.humidity.toFixed(2)} %`;
      }
      return 'N/A';
    });

    return {
      temperature,
      heatindex,
      humidity,
      tempHiChart,
      humidityChart,
      points,
      shift,
      CreateCharts
    };
  }
};
</script>

<style scoped>
.surface {
  background-color: var(--v-surface);
}
.highcharts-figure {
  margin: 0;
}
.mb-5 {
  margin-bottom: 5px;
}
.text-onPrimaryContainer {
  color: var(--v-on-primary-container);
}
.text-onTertiaryContainer {
  color: var(--v-on-tertiary-container);
}
.text-onSecondaryContainer {
  color: var(--v-on-secondary-container);
}
figure {
  border: 2px solid black;
}
</style>