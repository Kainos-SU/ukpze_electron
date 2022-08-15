<script setup lang="ts">
import { ref, onMounted } from "vue";
import { PortInfo } from "@serialport/bindings-interface";
import { SerialPort } from "serialport";
import { ipcRenderer } from "electron";

const portList = ref<Array<String>>([]);
const errorMsg = ref<String>("");
const updatePortList = () => {
    console.log("Updating Ports");
    SerialPort.list()
    .then((ports: Array<PortInfo>) => {
        portList.value = ports.map(port => port.path)
    })
    .catch((error: Error) => {
        portList.value = [];
        errorMsg.value = error.message;
    })
}

ipcRenderer.addListener("new-usb-device", updatePortList);
ipcRenderer.addListener("usb-device-remove", updatePortList);

onMounted(updatePortList);
</script>

<template>
    <template v-if="(!portList.length) && errorMsg === ''">
        No Ports Found
    </template>
    <template v-else-if="portList.length && errorMsg === ''">
        <ol>
            <li v-for="port in portList">{{ port }}</li>
        </ol>
    </template>
    <template v-else>{{ errorMsg }}</template>
    <button @click="updatePortList">Update Ports List</button>
</template>