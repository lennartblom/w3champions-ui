<template>
  <v-menu offset-x>
    <template v-slot:activator="{ on }">
      <v-btn tile v-on="on" style="background-color: transparent;">
        <v-icon style="margin-right: 5px;">mdi-controller-classic</v-icon>
        {{ gameModeName }}
      </v-btn>
    </template>
    <v-card>
      <v-card-text>
        <v-list>
          <v-list-item-content>
            <v-list-item-title>Select a gamemode:</v-list-item-title>
          </v-list-item-content>
        </v-list>
        <v-divider></v-divider>
        <v-list dense>
          <v-list-item
            v-for="mode in gameModes"
            :key="mode.gameMode"
            @click="selectGameMode(mode.gameMode)"
          >
            <v-list-item-content>
              <v-list-item-title>{{ mode.modeName }}</v-list-item-title>
            </v-list-item-content>
          </v-list-item>
        </v-list>
      </v-card-text>
    </v-card>
  </v-menu>
</template>

<script lang="ts">
import Vue from "vue";
import { Component, Prop } from "vue-property-decorator";

import { EGameMode } from "../../store/typings";

@Component({})
export default class GameModeSelect extends Vue {
  @Prop() gameMode?: EGameMode;

  get gameModes() {
    return [
      {
        modeName: this.$t(`gameModes.${EGameMode[EGameMode.GM_1ON1]}`),
        gameMode: EGameMode.GM_1ON1,
      },
      {
        modeName: this.$t(`gameModes.${EGameMode[EGameMode.GM_2ON2]}`),
        gameMode: EGameMode.GM_2ON2,
      },
      {
        modeName: this.$t(`gameModes.${EGameMode[EGameMode.GM_2ON2_AT]}`),
        gameMode: EGameMode.GM_2ON2_AT,
      },
      {
        modeName: this.$t(`gameModes.${EGameMode[EGameMode.GM_FFA]}`),
        gameMode: EGameMode.GM_FFA,
      },
    ];
  }

  get gameModeName() {
    if (!this.gameMode) {
      return "";
    }

    return this.gameModes.filter((g) => g.gameMode == this.gameMode)[0]
      .modeName;
  }

  public selectGameMode(gameMode: EGameMode) {
    this.$emit("gameModeChanged", gameMode);
  }
}
</script>

<style></style>
