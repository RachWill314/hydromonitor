<template>
    <v-container fluid class="d-flex justify-center">
        <v-row class="d-flex justify-center" style="max-width: 1200px;">
            <v-col class="d-flex align-center justify-center flex-column">
                <!-- Column 1 content -->
                <v-sheet color="surface" elevation="0" max-width="800" width="100%" class="mb-1 rounded-t-lg">
                    <v-card class="text-secondary mb-3 text-center" title="LED Controls" color="surface" subtitle="Recent settings"
                        variant="tonal" flat>
                    </v-card>
                </v-sheet>
                <v-sheet color="surface" elevation="0" max-width="800" width="100%" class="mb-1">
                    <v-card color="surface" variant="tonal" class="pt-5">
                        <v-slider class="pt-2 bg-surface" append-icon="mdi:mdi-car-light-high" density="compact"
                            thumb-size="16" color="secondary" label="Brightness" direction="horizontal" :min="0"
                            :max="250" :step="10" show-ticks thumb-label="always" v-model="led.brightness"></v-slider>
                    </v-card>
                </v-sheet>
                <v-sheet color="surface" elevation="0" max-width="800" width="100%"
                    class="mb-1 rounded-t-lg">
                    <v-card color="surface" variant="tonal" class="pt-5">
                        <v-slider 
                        class="pt-2 bg-surface" 
                        append-icon="mdi:mdi-led-on" 
                        density="compact" 
                        thumb-size="16"
                        color="secondary" 
                        label="LED Nodes" 
                        direction="horizontal" 
                        :min="1" :max="7" :step="1"
                        show-ticks thumb-label="always" 
                        v-model="led.leds"></v-slider>
                    </v-card>
                </v-sheet>
                <v-sheet color="surface" elevation="0" max-width="800" width="100%"
                    class="mb-1 pa-2 d-flex align-center justify-center" style="border: 1px solid;">
                    <v-progress-circular rotate="0" size="200" width="15" :model-value="led.leds * 15"><span
                            class="text-onSurface font-weight-bold">{{ led.leds }} LED(s)</span></v-progress-circular>
                </v-sheet>
            </v-col>
            <v-col class="d-flex align-center justify-center">
                <!-- Column 2 content -->
                <v-color-picker v-model="led.color"></v-color-picker>
            </v-col>
        </v-row>
    </v-container>
</template> // For HTML Code
<script setup>

import { useMqttStore } from '@/store/mqttStore'; // Import Mqtt Store
import { storeToRefs } from "pinia";
import { ref, onMounted, onBeforeUnmount, reactive, watch } from "vue";
// VARIABLES
const Mqtt = useMqttStore();
const { payload, payloadTopic } = storeToRefs(Mqtt);

const led = reactive({ "brightness": 255, "leds": 1, "color": { r: 255, g: 0, b: 255, a: 1 } });
let timer, ID = 1000;

onMounted(() => {
    // THIS FUNCTION IS CALLED AFTER THIS COMPONNT HAS BEEN MOUNTED
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

// WATCHERS
watch(led, (controls) => {
    clearTimeout(ID);
    ID = setTimeout(() => {
        const message = JSON.stringify({
            "type": "controls", "brightness": controls.brightness, "leds": controls.leds, "color":
                controls.color
        });
        Mqtt.publish("620155671_sub", message); // Publish to a topic subscribed to by the hardware
    }, 1000)
})

</script> // For JavaScript Code
<style></style> // For CSS Styling