<template>
  <v-card-text>
    <div v-if="hasNoClan && !isLoggedInPlayer">
      <v-row class="justify-center">
        <v-col class="text-center">
          <v-card-subtitle>
            This player is not part of a clan
          </v-card-subtitle>
        </v-col>
      </v-row>
    </div>
    <accept-invite-panel v-if="hasPendingInvite && isLoggedInPlayer" />
    <clan-creation-panel
      v-if="!hasPendingInvite && hasNoClan && isLoggedInPlayer"
    />
    <div v-if="!hasNoClan">
      <v-card-title class="justify-space-between">
        <span>{{ playersClan.clanName }} ({{ playersClan.clanId }})</span>
        <invite-player-modal v-if="loggedInPlayerIsShaman" />
      </v-card-title>
      <br />
      <br />
      <v-row>
        <v-col>
          <player-league :small-mode="true" :mode-stat="getStats(modeEnums.GM_1ON1)" />
        </v-col>
        <v-col>
          <player-league :small-mode="true" :mode-stat="getStats(modeEnums.GM_2ON2)" />
        </v-col>
        <v-col>
          <player-league :small-mode="true" :mode-stat="getStats(modeEnums.GM_2ON2_AT)" />
        </v-col>
        <v-col>
          <player-league :small-mode="true" :mode-stat="getStats(modeEnums.GM_FFA)" />
        </v-col>
      </v-row>
      <div v-if="playersClan.isSuccesfullyFounded">
        <table class="custom-table">
          <tr @click="gotToChiefTain">
            <td>
              <v-row class="justify-space-between align-center ma-0">
                <v-col class="pa-0">
                  <clan-role-icon :role="roleEnums.ChiefTain" />
                  <span
                    class="pointer"
                    @click="goToPlayer(playersClan.chiefTain)"
                  >
                    {{ playersClan.chiefTain.split("#")[0] }}
                  </span>
                  <league-icon
                    class="ml-4 mb-1"
                    :league="getLeagueOrder(playersClan.chiefTain)"
                  />
                </v-col>
              </v-row>
            </td>
          </tr>
        </table>
        <table class="custom-table">
          <tr
            v-for="member in shamans"
            :key="member"
            @click="goToPlayer(member)"
          >
            <td>
              <v-row class="justify-space-between align-center ma-0">
                <v-col class="pa-0">
                  <clan-role-icon :role="roleEnums.Shaman" />
                  <span class="pointer" @click="goToPlayer(member)">
                    {{ member.split("#")[0] }}
                  </span>
                  <league-icon
                    class="ml-4 mb-1"
                    :league="getLeagueOrder(member)"
                  />
                </v-col>
                <v-col class="text-right pa-0">
                  <member-management-menu
                    v-if="loggedInPlayerIsChiefTain"
                    :battle-tag="member"
                    :role="defineRole(member)"
                    :logged-in-user-role="loggedInRole"
                  />
                </v-col>
              </v-row>
            </td>
          </tr>
        </table>
        <table class="custom-table">
          <tr
            v-for="member in members"
            :key="member"
            @click="goToPlayer(member)"
          >
            <td>
              <v-row class="justify-space-between align-center ma-0">
                <v-col class="pa-0">
                  <clan-role-icon :role="roleEnums.Member" />
                  <span class="pointer" @click="goToPlayer(member)">
                    {{ member.split("#")[0] }}
                  </span>
                  <league-icon
                    class="ml-4 mb-1"
                    :league="getLeagueOrder(member)"
                  />
                </v-col>
                <v-col class="text-right pa-0">
                  <member-management-menu
                    v-if="loggedInPlayerIsShaman"
                    :battle-tag="member"
                    :role="defineRole(member)"
                    :logged-in-user-role="loggedInRole"
                  />
                </v-col>
              </v-row>
            </td>
          </tr>
        </table>
      </div>
      <div v-if="!playersClan.isSuccesfullyFounded">
        <v-card-title>
          Signees ({{ playersClan.foundingFathers.length }} / 7):
        </v-card-title>
        <table class="custom-table">
          <tr
            v-for="member in playersClan.foundingFathers"
            :key="member"
            @click="goToPlayer(member)"
          >
            <td>
              <span class="pointer" @click="goToPlayer(member)">
                {{ member.split("#")[0] }}
              </span>
            </td>
          </tr>
        </table>
      </div>
      <pending-invites-panel v-if="loggedInPlayerIsShaman" />
      <leave-clan-modal
        v-if="isLoggedInPlayer"
        :is-chieftain="loggedInPlayerIsChiefTain"
      />
      <delete-clan-modal v-if="loggedInPlayerIsChiefTain" />
    </div>
  </v-card-text>
</template>

<script lang="ts">
import Vue from "vue";
import { Component } from "vue-property-decorator";
import ClanCreationPanel from "@/components/clans/ClanCreationPanel.vue";
import InvitePlayerModal from "@/components/clans/InvitePlayerModal.vue";
import PendingInvitesPanel from "@/components/clans/PendingInvitesPanel.vue";
import AcceptInvitePanel from "@/components/clans/AcceptInvitePanel.vue";
import LeaveClanModal from "@/components/clans/LeaveClanModal.vue";
import MemberManagementMenu from "@/components/clans/MemberManagementMenu.vue";
import { EClanRole } from "@/store/clan/types";
import DeleteClanModal from "@/components/clans/DeleteClanModal.vue";
import { EGameMode } from "@/store/typings";
import LeagueIcon from "@/components/ladder/LeagueIcon.vue";
import PlayerAvatar from "@/components/player/PlayerAvatar.vue";
import ChatWindow from "@/components/chat/ChatWindow.vue";
import ChatUserList from "@/components/chat/ChatUserList.vue";
import ClanRoleIcon from "@/components/clans/ClanRoleIcon.vue";
import PlayerLeague from "@/components/player/PlayerLeague.vue";
import { ModeStat } from "@/store/player/types";

@Component({
  components: {
    PlayerLeague,
    ClanRoleIcon,
    ChatUserList,
    ChatWindow,
    PlayerAvatar,
    LeagueIcon,
    DeleteClanModal,
    MemberManagementMenu,
    LeaveClanModal,
    AcceptInvitePanel,
    PendingInvitesPanel,
    InvitePlayerModal,
    ClanCreationPanel,
  },
})
export default class ClanOverview extends Vue {
  public getLeagueOrder(battleTag: string) {
    return this.playersClan.ranks?.find(
      (r) => r.gameMode === EGameMode.GM_1ON1 && r.id.includes(battleTag)
    )?.leagueOrder;
  }

  public getStats(mode: EGameMode) {
    const games = this.playersClan.ranks?.filter((r) => r.gameMode === mode);
    const players = games.map((l) => l.player);
    if (players.length === 0) return { games: 0, gameMode: mode } as ModeStat;

    const reduced = players.reduce(
      (a, b) => ({
        wins: a.wins + b.wins,
        losses: a.losses + b.losses,
        gameMode: mode,
        games: a.games + b.games,
        rank: 5,
        leagueOrder: 5,
      }),
      {
        wins: 0,
        losses: 0,
        gameMode: mode,
        games: 0,
        rank: 5,
        leagueOrder: 5,
      }
    );

    const order = this.playersClan.ranks.reduce((a, b) => ({ leagueOrder: a.leagueOrder + b.leagueOrder }), { leagueOrder: 0 })
    reduced.leagueOrder = Math.round(order.leagueOrder / games.length);

    return reduced;
  }

  get roleEnums() {
    return EClanRole;
  }

  get modeEnums() {
    return EGameMode;
  }

  get clanValidationError() {
    return this.$store.direct.state.clan.clanValidationError;
  }

  get hasPendingInvite() {
    return this.$store.direct.state.clan.selectedMemberShip
      ?.pendingInviteFromClan;
  }

  get searchPlayers() {
    return this.$store.direct.state.clan.searchPlayers;
  }

  public gotToChiefTain() {
    this.goToPlayer(this.playersClan.chiefTain);
  }

  public defineRole(member: string) {
    if (member === this.playersClan.chiefTain) return EClanRole.ChiefTain;
    if (this.playersClan.shamans.find((s) => s === member))
      return EClanRole.Shaman;
    return EClanRole.Member;
  }

  get loggedInRole() {
    return this.defineRole(this.verifiedBtag);
  }

  get loggedInPlayerIsChiefTain() {
    return this.playersClan.chiefTain === this.verifiedBtag;
  }

  get loggedInPlayerIsShaman() {
    return (
      this.playersClan.shamans.find((s) => s === this.verifiedBtag) ||
      this.loggedInPlayerIsChiefTain
    );
  }

  public goToPlayer(battleTag: string) {
    this.$router.push({ path: "/player/" + encodeURIComponent(battleTag) });
  }

  get verifiedBtag() {
    return this.$store.direct.state.oauth.blizzardVerifiedBtag;
  }

  get hasNoClan() {
    return !this.playersClan?.clanId;
  }

  get isLoggedInPlayer() {
    return this.verifiedBtag === this.selectedPlayer;
  }

  get selectedPlayer() {
    return this.$store.direct.state.player.battleTag;
  }

  get playersClan() {
    return this.$store.direct.state.clan.playersClan;
  }

  get shamans() {
    return this.$store.direct.state.clan.playersClan.shamans;
  }

  get members() {
    return this.$store.direct.state.clan.playersClan.members;
  }

  async mounted() {
    await this.$store.direct.dispatch.clan.retrievePlayersMembership();
    await this.$store.direct.dispatch.clan.retrievePlayersClan();
  }
}
</script>
