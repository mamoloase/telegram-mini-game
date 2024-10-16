<template>
    <section class="mt-4">
        <div class="flex-center flex-column gap-2">
            <span class="description-color text-sm text-shadow">Leaderboard</span>

            <div class="flex-center gap-2">
                <div class="flex-center">
                    <LeaderboardIcon width="40px" height="40px" />
                </div>
                <span class="text-lg text-color text-shadow">On Top</span>
            </div>
        </div>
    </section>


    <section class="mt-5">

        <div class="box-shadow bg-card p-3 rounded-2">
            <div v-if="showPlayers" class="w-full flex-center flex-column">
                <SearchBox @change="changeQueryData" title="Players" class="w-full" />

                <div class="w-full flex-center flex-column mt-3">
                    <div v-if="members && members.length == 0" class="flex-center py-3">
                        <span class="text-color text-md text-shadow">Can not found Players</span>
                    </div>

                    <UserListCard :publicView="true" v-if="members" v-for="(   user, index   ) in members   "
                        :key="index" :user="user" />
                    <div v-else v-for="(x, y) in [0, 0, 0]" :key="y"
                        class="w-full d-flex align-items-center justify-content-between my-2">
                        <div class="flex-center gap-2">
                            <SkeletonLoader width="40px" height="40px" />
                            <div class="d-flex flex-column gap-2">
                                <SkeletonLoader width="40px" height="15px" />
                                <SkeletonLoader width="70px" height="15px" />
                            </div>
                        </div>
                        <SkeletonLoader width="50px" height="15px" />
                    </div>
                </div>
            </div>
            <div v-else class="w-full flex-center flex-column">

                <div class="w-full flex-center">
                    <div class="d-flex align-items-center grow-1">
                        <span class="description-color text-md text-shadow">
                            Referral Reward
                        </span>
                    </div>
                </div>
                <div class="w-full flex-center mt-3">
                    <div class="d-flex align-items-center gap-2 grow-1">
                        <div class="flex-center mw-content border-light box-shadow p-2 rounded-circle">
                            <img width="30px" :src="$filters.serverLink(selectedLeague?.image ?? '')" alt="">
                        </div>
                        <span class="text-color text-md text-shadow">{{ selectedLeague?.name ?? "League" }}</span>
                    </div>
                    <div class="d-flex align-items-center">
                        <span class="description-color text-md text-shadow">
                            + {{ $filters.formatNumber(selectedLeague?.referralReward ?? 0) }}
                        </span>
                    </div>
                </div>

                <div class="w-full flex-center mt-4">
                    <div class="d-flex align-items-center grow-1">
                        <span class="description-color text-md text-shadow">
                            Lucky Rewards
                        </span>
                    </div>
                </div>
                <div v-if="selectedLeague" class="d-flex align-items-center mt-3 w-full  flex-wrap gap-2">
                    <div class="flex-center flex-column gap-2 p-3 px-4 rounded-2 box-shadow grow-1"
                        v-for="(reward, index) in selectedLeague.rewards" :key="index">
                        <div class="d-flex align-items-center gap-2">
                            <div class="flex-center mw-content">
                                <GhostIcon v-if="reward.type == RewardType.Null" width="25px" height="25px" />
                                <CoinIcon v-else-if="reward.type == RewardType.Coin" width="25px" height="25px" />
                                <TonIcon v-else-if="reward.type == RewardType.Ton" width="25px" height="25px" />
                            </div>
                        </div>
                        <div class="d-flex align-items-center">
                            <span class="text-color text-md text-shadow">
                                +{{ reward.type == RewardType.Null ? "0" : reward.reward }}
                            </span>
                        </div>
                    </div>
                </div>


            </div>
        </div>
    </section>
</template>
<style scoped>
.border-botton {
    border-bottom: 1px solid rgba(var(--border-color), 0.5);
}
</style>
<script lang="ts" setup>
import { ref } from 'vue';
import { useStore } from '@/store';
import UserModel from '@/models/userModel';
import LeagueModel from '@/models/leagueModel';
import SearchBox from '@/components/inputs/SearchBox.vue';
import UserListCard from '@/components/cards/UserListCard.vue';
import SkeletonLoader from '@/components/utilities/SkeletonLoader.vue';
import LeagueService from '@/services/leagueService';
import CoinIcon from '@/components/icons/filled/CoinIcon.vue';
import ProgressLine from '@/components/progresses/ProgressLine.vue';
import Line from '@/components/utilities/Line.vue';
import { RewardType } from '@/models/rewardModel';
import GiftIcon from '@/components/icons/filled/GiftIcon.vue';
import GhostIcon from '@/components/icons/filled/GhostIcon.vue';
import TonIcon from '@/components/icons/filled/TonIcon.vue';
import LeaderboardIcon from '@/components/icons/filled/LeaderboardIcon.vue';

const store = useStore();

const query = ref<string>("");
const showPlayers = ref<boolean>(true);

const members = ref<Array<UserModel> | null>(null);
const selectedLeague = ref<LeagueModel | null>(store.getters.getLeague);

const showBoxPlayers = (status: boolean) => {
    if (status == showPlayers.value) return;
    showPlayers.value = status;
}

const getLeagueMembers = () => {
    if (!selectedLeague.value) return;

    new LeagueService().getMembers(selectedLeague.value.id, query.value)
        .then(result => {
            members.value = result?.result ?? []
        })
        .catch(except => { })
}

const changeQueryData = (q: string) => {
    query.value = q;
    const currentValue = query.value;

    if (q.length == 0) {
        members.value = new Array<UserModel>();
        return;
    }
    if (q.length < 3) return;

    members.value = null;

    setTimeout(() => {
        if (query.value == currentValue) {
            members.value = null;
            getLeagueMembers();
        }
    }, 3000);
};
const chenageLeague = (league: LeagueModel) => {
    if (league == selectedLeague.value) return;

    selectedLeague.value = league;
    getLeagueMembers()
}
getLeagueMembers();
</script>