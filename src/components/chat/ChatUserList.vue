<template>
  <div>
    <v-card class="pt-4" style="height: 630px;">
      <div class="overflow-y-auto">
        <v-list-item
          style="min-height: 25px !important;"
          v-for="user in otherUsers"
          :key="user.battleTag"
          @click="openProfile(user.battleTag)"
        >
          {{ user.name }}
        </v-list-item>
      </div>
    </v-card>
  </div>
</template>
<script lang="ts">
import Vue from "vue";
import { Component } from "vue-property-decorator";

@Component({})
export default class ChatUserList extends Vue {
  get otherUsers() {
    return this.$store.direct.state.chat.otherUsers;
  }

  public openProfile(battleTag: string) {
    this.$router.push({ path: `/player/${encodeURIComponent(battleTag)}` });
  }
}
</script>
