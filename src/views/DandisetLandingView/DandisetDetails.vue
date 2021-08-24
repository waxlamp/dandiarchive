<template>
  <v-card
    v-if="currentDandiset"
    height="100%"
    class="px-3 py-1"
  >
    <v-row
      v-if="contactName"
      no-gutters
      :class="rowClasses"
    >
      <v-icon>mdi-account</v-icon>
      <span :class="labelClasses">Contact</span>
      <span :class="itemClasses">{{ contactName }}</span>
    </v-row>
    <v-row
      no-gutters
      :class="rowClasses"
    >
      <v-icon>mdi-calendar</v-icon>
      <span :class="labelClasses">Created on</span>
      <span :class="itemClasses">{{ created }}</span>
    </v-row>
    <v-row
      no-gutters
      :class="rowClasses"
    >
      <v-icon>mdi-update</v-icon>
      <span :class="labelClasses">Last modified</span>
      <span :class="itemClasses">{{ lastModified }}</span>
    </v-row>

    <v-divider class="my-2" />

    <v-row :class="`${rowClasses} px-2`">
      <span :class="labelClasses">Identifier</span>
      <span :class="itemClasses">{{ currentDandiset.dandiset.identifier }}</span>
    </v-row>

    <template v-if="stats">
      <v-row :class="`${rowClasses} px-2`">
        <v-col
          cols="auto"
          class="text--secondary mx-2 pa-0 py-1"
        >
          <v-icon color="primary">
            mdi-file
          </v-icon>
          {{ stats.asset_count }}
        </v-col>
        <v-col
          class="text--secondary mx-2 pa-0 py-1"
          cols="auto"
        >
          <v-icon color="primary">
            mdi-server
          </v-icon>
          {{ formattedSize }}
        </v-col>
      </v-row>
    </template>

    <v-divider class="mt-2 px-0 mx-0" />

    <v-row>
      <v-col>
        <v-card
          color="grey lighten-3"
          class="mx-2"
          flat
          tile
        >
          <v-row
            no-gutters
            class="py-2"
          >
            <v-col cols="11">
              <v-icon
                class="mx-2"
                color="primary"
              >
                mdi-source-branch
              </v-icon>
              <span :class="`${itemClasses} text-capitalize`"> {{ currentVersion }} </span>
            </v-col>
            <v-col v-if="draftDandiset">
              <v-tooltip top>
                <template #activator="{ on }">
                  <v-icon
                    small
                    color="grey darken-1"
                    v-on="on"
                  >
                    mdi-help-circle
                  </v-icon>
                </template>
                <span>This is a version of your dandiset that contains unpublished changes.</span>
              </v-tooltip>
            </v-col>
          </v-row>
        </v-card>
      </v-col>
    </v-row>

    <v-divider class="my-2 px-0 mx-0" />

    <v-row
      no-gutters
      :class="rowClasses"
    >
      <v-col>
        <span :class="labelClasses">Ownership</span>
      </v-col>
      <v-col cols="auto">
        <v-dialog
          v-model="ownerDialog"
          width="50%"
        >
          <template #activator="{ on }">
            <v-tooltip
              :disabled="!manageOwnersDisabled"
              left
            >
              <template #activator="{ on: tooltipOn }">
                <div v-on="tooltipOn">
                  <v-btn
                    id="manage"
                    color="primary"
                    x-small
                    text
                    :disabled="manageOwnersDisabled"
                    v-on="on"
                  >
                    <v-icon
                      x-small
                      class="pr-1"
                    >
                      mdi-lock
                    </v-icon>
                    Manage
                  </v-btn>
                </div>
              </template>
              <template v-if="loggedIn">
                You must be an owner to manage ownership.
              </template>
              <template v-else>
                You must be logged in to manage ownership.
              </template>
            </v-tooltip>
          </template>
          <DandisetOwnersDialog
            :key="ownerDialogKey"
            :owners="owners"
            @close="ownerDialog = false"
          />
        </v-dialog>
      </v-col>
    </v-row>

    <!-- TODO: Make chips wrap, instead of pushing whole card wide -->
    <v-row :class="rowClasses">
      <v-col cols="12">
        <v-chip
          v-for="owner in limitedOwners"
          :key="owner.id /* TODO remove this */ || owner.name || owner.username"
          color="light-blue lighten-4"
          text-color="light-blue darken-3"
          class="font-weight-medium ma-1"
        >
          {{ owner.login /* TODO remove this */ || owner.name || owner.username }}
        </v-chip>
        <span
          v-if="numExtraOwners"
          class="ml-1 text--secondary"
        >
          +{{ numExtraOwners }} more...
        </span>
      </v-col>
    </v-row>

    <v-row>
      <v-col class="pa-0">
        <v-card
          color="grey lighten-2"
          tile
          flat
        >
          <div class="py-1 px-3">
            Versions
          </div>
        </v-card>
      </v-col>
    </v-row>
    <v-row>
      <v-timeline dense>
        <v-timeline-item
          v-for="version in versions"
          :key="version.version"
          small
          right
          :color="timelineVersionItemColor(version)"
        >
          <v-btn
            text
            class="font-weight-medium"
            @click="setVersion(version)"
          >
            {{ version.version }}
          </v-btn>
        </v-timeline-item>
      </v-timeline>
    </v-row>
  </v-card>
</template>

<script lang="ts">
import {
  defineComponent, ref, computed, watch, Ref, ComputedRef,
} from '@vue/composition-api';

import { loggedIn as loggedInFunc, user as userFunc } from '@/rest';
import { User, Version } from '@/types';
import moment from 'moment';
import filesize from 'filesize';

import { draftVersion } from '@/utils/constants';
import { RawLocation } from 'vue-router';
import DandisetOwnersDialog from './DandisetOwnersDialog.vue';

export default defineComponent({
  name: 'DandisetDetails',
  components: {
    DandisetOwnersDialog,
  },
  setup(props, ctx) {
    // constants used by the template
    const rowClasses = 'my-1';
    const labelClasses = 'mx-2 text--secondary';
    const itemClasses = 'font-weight-medium';
    const ownerDialog = ref(false);
    const ownerDialogKey = ref(0);

    const store = ctx.root.$store;

    const currentDandiset: ComputedRef<Version> = computed(
      () => store.state.dandiset.publishDandiset,
    );
    const owners: ComputedRef<User[]> = computed(() => store.state.dandiset.owners);
    const versions: ComputedRef<Version[]> = computed(
      () => store.state.dandiset.versions,
    );
    const currentVersion: ComputedRef<Version> = computed(
      () => store.getters['dandiset/version'],
    );

    const stats: Ref<any> = ref(null);
    watch(currentDandiset, async () => {
      if (currentDandiset.value) {
        const { asset_count, size } = currentDandiset.value;
        stats.value = { asset_count, size };
      }
    }, { immediate: true });

    function formatDateTime(datetimeStr: string): string {
      const datetime = moment(datetimeStr);
      const date = datetime.format('LL');
      const time = datetime.format('hh:mm A');

      return `${date} at ${time}`;
    }

    const user: ComputedRef<User|null> = computed(userFunc);
    const loggedIn = computed(loggedInFunc);
    const created = computed(() => formatDateTime(currentDandiset.value.created));
    const lastModified = computed(() => formatDateTime(currentDandiset.value.modified));
    const contactName = computed(() => currentDandiset.value.dandiset.contact_person);
    const draftDandiset = computed(() => currentVersion.value.version === draftVersion);
    const manageOwnersDisabled: ComputedRef<boolean> = computed(() => {
      if (!loggedIn || !owners.value) return true;
      return !owners.value.find((owner: User) => owner.username === user.value?.username);
    });
    const limitedOwners: ComputedRef<User[]> = computed(() => {
      if (!owners.value) return [];
      return owners.value.slice(0, 5);
    });
    const numExtraOwners: ComputedRef<number> = computed(() => {
      if (!owners.value) return 0;
      return owners.value.length - limitedOwners.value.length;
    });
    const formattedSize: ComputedRef<string|undefined> = computed(() => {
      if (!stats.value) {
        return undefined;
      }
      return filesize(stats.value.size, { round: 1, base: 10, standard: 'iec' });
    });

    function setVersion({ version: newVersion }: any) {
      const version = newVersion || draftVersion;

      if (ctx.root.$route.params.version !== version) {
        ctx.root.$router.replace({
          ...ctx.root.$route,
          params: {
            ...ctx.root.$route.params,
            version,
          },
        } as RawLocation);
      }
    }

    function timelineVersionItemColor({ version }: any): string {
      if (currentDandiset.value && version === currentDandiset.value.version) { return 'primary'; }
      if (!currentDandiset.value && version === draftVersion) {
        return 'amber darken-4';
      }

      return 'grey';
    }

    return {
      rowClasses,
      labelClasses,
      itemClasses,
      ownerDialog,
      ownerDialogKey,
      currentDandiset,
      owners,
      versions,
      currentVersion,
      stats,
      user,
      loggedIn,
      created,
      lastModified,
      contactName,
      draftDandiset,
      manageOwnersDisabled,
      limitedOwners,
      numExtraOwners,
      formattedSize,
      setVersion,
      timelineVersionItemColor,
    };
  },
});
</script>
