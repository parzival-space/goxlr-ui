<template>
  <div class="wrapper">
  <div class="buttonList">
    <div>
      <div class="label">Select Device</div>

      <!-- If we've never connected before, and we're not connected now.. -->
      <div v-if="!hasConnected() && !isConnected()">
        <div class="no-device">Attempting to Connect to the GoXLR Utility..</div>
      </div>

      <!-- We *HAVE* connected before, but we're not connected now.. -->
      <div v-else-if="hasConnected() && !isConnected()">
        <div class="no-device">Unable to connect to the GoXLR Utility, please check it's running.<br /><br /> This page will automatically try to reconnect..</div>
      </div>

      <!-- We should be connected here! -->
      <div v-else>
        <div class="buttonHolder" v-if="deviceCount > 0">
          <Button v-for="(device, key) in getMixers()" :key=key :button-id=key :is-active=false
                  :label="getLabel2(key, device)" @button-pressed="setDevice(key)"/>
        </div>
        <div v-else class="no-device">No GoXLR Devices Found</div>
      </div>
    </div>
  </div>
  <div v-if="isConnected() && hasConfig()" class="buttonList" style="width: 170px">
    <div class="buttonHolder" style="width: 170px; padding-top: 25px; overflow-y: initial">
      <SettingsButton />
    </div>
  </div>
  </div>
</template>

<script>

import Button from "@/components/buttons/Button.vue";
import {store} from "@/store";
import SettingsButton from "@/components/sections/system/modals/SettingsButton.vue";

export default {
  name: "DeviceSelector",
  components: {SettingsButton, Button},
  data() {
    return {
      devices: [],
    }
  },

  computed: {
    deviceCount() {
      return store.getDeviceCount();
    }
  },

  watch: {
    // This code probably isn't needed, but is for when a GoXLR suddenly appears in the data.
    deviceCount(newCount, oldCount) {
      if (newCount === 1 && oldCount === 0) {
        store.setActiveSerial(Object.keys(this.getMixers())[0]);
      }
    }
  },

  methods: {
    hasConnected() {
      return store.hasConnected();
    },

    isConnected() {
      return store.isConnected();
    },

    hasConfig() {
      return store.getConfig() !== undefined;
    },

    getMixers() {
      return store.status.mixers;
    },

    setDevice(serial) {
      store.setActiveSerial(serial);
    },

    getLabel2(serial, device) {
      return "[" + serial + "] GoXLR " + device.hardware.device_type + " connected to USB bus " + device.hardware.usb_device.bus_number + " address " + device.hardware.usb_device.address;
    },
  },

  created() {
    if (this.deviceCount === 1) {
      store.setActiveSerial(Object.keys(this.getMixers())[0]);
    }
  }
}
</script>

<style scoped>
.wrapper {
  text-align: center;
  display: flex;
  justify-content: center;
  align-items: center;
}

.buttonList {
  height: 220px;
  width: 700px;
  margin: 3px;
  background-color: #353937;
}

.buttonList:not(:last-child) {
  margin-right: 20px;
}

.buttonHolder {
  height: 170px;
  width: 700px;

  box-sizing: border-box;

  overflow-y: auto;

}

.buttonHolder::-webkit-scrollbar {
  width: 3px;
}

.buttonHolder::-webkit-scrollbar-track {
  background-color: transparent;
}

.buttonHolder::-webkit-scrollbar-thumb {
  background-color: #dfdfdf;
  border-radius: 3px;
}

.label {
  width: 680px;
  padding: 10px;
  color: #fff;
  background-color: #3b413f;

  text-transform: uppercase;

  margin-bottom: 8px;
}

.no-device {
  color: #fff;
}

</style>
