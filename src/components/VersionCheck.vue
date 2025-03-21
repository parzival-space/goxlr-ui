<template>
  <div v-if="hasVersion()" class="version">
    GoXLR Utility v{{getVersion()}}
    <span v-if="outdated()"> - <a :href="release_path" target="_blank"> Update Available</a></span>
    <span v-if="firmware_outdated()"> - <span class="click" @click="$refs.firmware_modal.openModal(undefined, undefined)">Firmware Update Available</span></span>
  </div>
  <div v-if="incompatibleDriver()" class="warning-wrap">
      <a class="warning" href="https://mediadl.musictribe.com/media/PLM/sftp/incoming/hybris/import/FirmwareAssets/GOXLR/LiveTestArea/driverRepair/TC-Helicon_GoXLR_Driver-5.68.0.zip" target="_blank">
        The installed drivers are incompatible with this firmware, click here to download the latest driver.
      </a>
  </div>

  <AccessibleModal width="630px" ref="firmware_modal" id="firmware_modal" :show_footer="true">
    <template v-slot:title>Firmware Update Available</template>
    <template v-slot:default>
      A firmware update (Version {{getFirmwareVersion()}}) is available for your device.<br /><br />

      At this point in time the GoXLR Utility is unable to perform a firmware update. Please plug your device
      into a Windows based machine and run the official GoXLR Application to receive the latest firmware.<br /><br />

      Once completed, re-launch the GoXLR Utility to continue using your GoXLR.
    </template>
  </AccessibleModal>
</template>

<script>
import {store} from "@/store";
import {driverPreVOD, firmwareSupportsMix2, isDeviceMini, versionNewerOrEqualTo} from "@/util/util";
import AccessibleModal from "@/components/design/modal/AccessibleModal.vue";

export default {
  name: "VersionCheck",
  components: {AccessibleModal},

  data: function () {
    return {
      version: undefined,
      release_path: "#",
    }
  },

  methods: {
    getLatest() {
      fetch(this.getPath())
          .then(response => {
            if (response.status !== 200) {
              return undefined;
            }
            return response.json()
          })
          .then(data => {
            if (data === undefined) {
              return;
            }
            fetch(data[0].url)
                .then(response => response.json())
                .then(data => {
                  this.version = data.tag_name.substring(1);
                  this.release_path = data.html_url;
                })
          });
    },

    hasVersion() {
      return (store.daemonVersion() !== undefined);
    },

    getVersion() {
      return store.daemonVersion();
    },

    getFirmwareVersion() {
      if (store.getActiveDevice() === undefined) {
        return false;
      }

      if (store.getConfig() === undefined) {
        return false;
      }

      if (store.getConfig().latest_firmware !== undefined && store.getConfig().latest_firmware !== null) {
        let latest = isDeviceMini() ? store.getConfig().latest_firmware.Mini : store.getConfig().latest_firmware.Full;
        if (latest === undefined || latest === null) {
          return false;
        }

        return latest.version.join(".");
      }
      return true;
    },

    outdated() {
      if (store.daemonVersion() === undefined) {
        return false;
      }
      return this.isOutdated(store.daemonVersion(), this.version);
    },

    firmware_outdated() {
      if (store.getActiveDevice() === undefined) {
        return false;
      }

      if (store.getConfig() === null || store.getConfig() === undefined) {
        return false;
      }

      if (store.getConfig().latest_firmware !== undefined && store.getConfig().latest_firmware !== null) {
        let latest = isDeviceMini() ? store.getConfig().latest_firmware.Mini : store.getConfig().latest_firmware.Full;
        if (latest === undefined || latest === null) {
          return false;
        }

        let current = store.getActiveDevice().hardware.versions.firmware;
        return (versionNewerOrEqualTo(latest, current) && !versionNewerOrEqualTo(current, latest));
      }

      // Fail Safe if versions are missing..
      return false;
    },

    incompatibleDriver() {
      return driverPreVOD() && firmwareSupportsMix2();
    },

    getPath() {
      return "https://api.github.com/repos/GoXLR-on-Linux/goxlr-utility/releases";
    },

    isOutdated(base_version, match_version) {
      if (match_version === undefined) {
        // If we can't map the current version, assume they match to avoid bugging the user.
        return false;
      }

      let base = base_version.split(".");
      let match = match_version.split(".");

      // This isn't perfect, but it doesn't need to be. If the util is reporting a different version from github, there
      // has likely been an update, we'll deal with edge cases separately..
      if (base[0] !== match[0]) {
        return true;
      }

      if (base[1] !== match[1]) {
        return true;
      }

      if (base[2] !== undefined) {
        if (match[2] !== undefined) {
          if (base[2] !== match[2]) {
            return true;
          }
        } else {
          return true;
        }
      }

      if (base[3] !== undefined) {
        if (match[3] !== undefined) {
          if (base[3] !== match[3]) {
            return true;
          }
        } else {
          return true;
        }
      }
      return false;
    }
  },

  mounted() {
    this.getLatest();
  }
}
</script>

<style scoped>
 .version {
   font-family: sans-serif;
   text-align: center;
   color: #444444;
   margin-bottom: 8px;
   padding: 3px;
 }

 .version a {
   color: #555555;
 }

 .version span.click {
   color: #555555;
   cursor: pointer;
   text-decoration: underline;
 }

 .warning-wrap {
   text-align: center;
 }

 .warning {
   margin: auto;
   background-color: #370000;
   border: 1px solid #6e0000;
   color: #8e8e8e;
   font-weight: bold;
   padding: 6px;
   text-align: center;
 }

 .warning a {
   color: #717171;
   text-decoration: none;
 }
</style>
