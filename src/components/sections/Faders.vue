<template>
  <GroupContainer :title="$t('message.configuration.faders.title')">
    <RadioSelection :title="$t('message.configuration.faders.channelTitle')" group="faders_channel"
                    :options="getFaderOptions()" :selected="activeChannel" @selection-changed="channelChanged" />

    <RadioSelection :title="$t('message.configuration.faders.sourceTitle')" group="faders_source"
                    :options="getSourceOptions()" :selected="getActiveSource()" @selection-changed="sourceChanged"
                    :label="$t('message.configuration.faders.sourceLabel', { channel: getActiveChannelName() })" />

    <RadioSelection :title="$t('message.configuration.muteBehaviourTitle')" group="faders_mute"
                    :options="getMuteBehaviour()" :selected="getActiveMuteBehaviour()"
                    @selection-changed="muteFunctionChanged"
                    :label="$t('message.configuration.muteBehaviourLabel', { channel: getActiveChannelName() })" />
  </GroupContainer>
</template>

<script>
import { store } from "@/store";
import { websocket } from "@/util/sockets";
import RadioSelection from "@/components/lists/RadioSelection.vue";
import GroupContainer from "@/components/containers/GroupContainer.vue";

export default {
  /**
   * Everything here focuses around the 'Channel' input,
   */

  emits: ["on-fader-channel-change"],
  components: { GroupContainer, RadioSelection },
  name: "MicFaders",

  data() {
    return {
      activeChannel: "A",
    }
  },

  methods: {
    getFaderOptions() {
      return [
        { id: "A", label: this.$t('message.faders.A') },
        { id: "B", label: this.$t('message.faders.B') },
        { id: "C", label: this.$t('message.faders.C') },
        { id: "D", label: this.$t('message.faders.D') }
      ];
    },

    getSourceOptions() {
      return [
        { id: "Mic", label: this.$t('message.channels.Mic') },
        { id: "Chat", label: this.$t('message.channels.Chat') },
        { id: "Music", label: this.$t('message.channels.Music') },
        { id: "Game", label: this.$t('message.channels.Game') },
        { id: "Console", label: this.$t('message.channels.Console') },
        { id: "LineIn", label: this.$t('message.channels.LineIn') },
        { id: "System", label: this.$t('message.channels.System') },
        { id: "Sample", label: this.$t('message.channels.Sample') },
        { id: "Headphones", label: this.$t('message.channels.Headphones') },
        { id: "LineOut", label: this.$t('message.channels.LineOut') }
      ];
    },

    getMuteBehaviours() {
      return [
        { id: "All", label: this.$t('message.configuration.mute_behaviour.all') },
        { id: "ToStream", label: this.$t('message.configuration.mute_behaviour.stream') },
        { id: "ToVoiceChat", label: this.$t('message.configuration.mute_behaviour.chatMic') },
        { id: "ToPhones", label: this.$t('message.configuration.mute_behaviour.headphones') },
        { id: "ToLineOut", label: this.$t('message.configuration.mute_behaviour.lineOut') },
      ];
    },

    channelChanged: function (id) {
      this.activeChannel = id;
      this.$emit("on-fader-channel-change", id);
    },

    sourceChanged: function (channelName) {
      let serial = store.getActiveSerial();
      let fader = this.activeChannel;

      let command = {
        "SetFader": [fader, channelName]
      }

      websocket.send_command(serial, command);
      store.getActiveDevice().fader_status[this.activeChannel].channel = channelName;

      if (this.isMuteFunctionDisabled(this.getActiveMuteBehaviour())) {
        this.muteFunctionChanged("All")
      }
    },

    muteFunctionChanged(mute_function) {
      let serial = store.getActiveSerial();
      let fader = this.activeChannel;

      // Check the make sure this combination is allowed..


      // Build the Command..
      let command = {
        "SetFaderMuteFunction": [
          fader,
          mute_function
        ]
      }

      websocket.send_command(serial, command);
    },

    updateDisabledMuteFunctions(source, behaviours) {
      for (let mute_option of behaviours) {
        if (source === "Chat" && mute_option.id === "ToVoiceChat") {
          mute_option.disabled = true;
          continue;
        }

        if ((source === "Headphones" || source === "LineOut") && (mute_option.id !== "All")) {
          mute_option.disabled = true;
          continue;
        }
        mute_option.disabled = false;
      }
      return behaviours;
    },

    isMuteFunctionDisabled: function (muteFunction) {
      // According to the GoXLR UI, The Voice Chat mute button can't mute to voice chat..
      if (this.getActiveSource() === "Chat") {
        if (muteFunction === "ToVoiceChat") {
          return true;
        }
      }

      // The Headphone and Line Out channels can only mute to 'All'.
      if (this.getActiveSource() === "Headphones" || this.getActiveSource() === "LineOut") {
        if (muteFunction !== "All") {
          return true;
        }
      }
    },

    getActiveSource: function () {
      return store.getActiveDevice().fader_status[this.activeChannel].channel;
    },

    getActiveMuteBehaviour: function () {
      return store.getActiveDevice().fader_status[this.activeChannel].mute_type;
    },

    getMuteBehaviour: function () {
      return this.updateDisabledMuteFunctions(this.getActiveSource(), this.getMuteBehaviours());
    },
    getActiveChannelName: function () {
      return this.getFaderOptions().find((option) => option.id === this.activeChannel).label;
    },
  },
}
</script>

<style scoped></style>
