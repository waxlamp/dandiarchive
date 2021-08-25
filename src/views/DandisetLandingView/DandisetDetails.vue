<template>
  <div>
    <v-card
      v-if="currentDandiset"
      height="100%"
      class="px-3 py-1 mb-3"
      outlined
    >
      <v-card-subtitle>
        Dandiset Actions
      </v-card-subtitle>

      <!-- Download, Shareable Link, and Cite As buttons -->
      <div class="mb-4">
        <v-row no-gutters>
          <DownloadDialog>
            <template
              #activator="{ on }"
              class="justify-left"
            >
              <v-btn
                id="download"
                outlined
                style="width: 100%"
                v-on="on"
              >
                <v-icon
                  color="primary"
                  left
                >
                  mdi-download
                </v-icon>
                Download
                <v-spacer />
                <v-icon right>
                  mdi-chevron-down
                </v-icon>
              </v-btn>
            </template>
          </DownloadDialog>
        </v-row>
        <v-row no-gutters>
          <ShareableLinkDialog>
            <template
              #activator="{ on }"
              class="justify-left"
            >
              <v-btn
                id="download"
                outlined
                style="width: 100%"
                v-on="on"
              >
                <v-icon
                  color="primary"
                  left
                >
                  mdi-link
                </v-icon>
                Shareable Link
                <v-spacer />
                <v-icon right>
                  mdi-chevron-down
                </v-icon>
              </v-btn>
            </template>
          </ShareableLinkDialog>
          <CiteAsDialog>
            <template
              #activator="{ on }"
              class="justify-left"
            >
              <v-btn
                id="download"
                outlined
                style="width: 100%"
                v-on="on"
              >
                <v-icon
                  color="primary"
                  left
                >
                  mdi-format-quote-close
                </v-icon>
                Cite As
                <v-spacer />
                <v-icon right>
                  mdi-chevron-down
                </v-icon>
              </v-btn>
            </template>
          </CiteAsDialog>
        </v-row>
      </div>

      <!-- Files and Metadata buttons -->
      <div>
        <v-row no-gutters>
          <v-btn
            outlined
            style="width: 100%"
            v-on="on"
          >
            <v-icon
              left
              color="primary"
            >
              mdi-folder
            </v-icon>
            Files
            <v-spacer />
          </v-btn>
        </v-row>
        <v-row no-gutters>
          <v-btn
            outlined
            style="width: 100%"
            v-on="on"
          >
            <v-icon
              left
              color="primary"
            >
              mdi-folder
            </v-icon>
            Metadata
            <v-spacer />
          </v-btn>
        </v-row>
      </div>
    </v-card>
    <v-card
      v-if="currentDandiset"
      height="100%"
      class="px-3 py-1"
      outlined
    >
      <v-row
        no-gutters
        :class="rowClasses"
      >
        <v-card-subtitle>
          Ownership
        </v-card-subtitle>
        <DandisetOwners
          v-if="owners"
          :key="ownerDialogKey"
          :owners="owners"
          @close="ownerDialog = false"
        />
      </v-row>
      <!-- <v-row>
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
    </v-card> -->
    </v-card>
  </div>
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
import DandisetOwners from './DandisetOwners.vue';
import DownloadDialog from './DownloadDialog.vue';
import ShareableLinkDialog from './ShareableLinkDialog.vue';
import CiteAsDialog from './CiteAsDialog.vue';

export default defineComponent({
  name: 'DandisetDetails',
  components: {
    CiteAsDialog,
    DandisetOwners,
    DownloadDialog,
    ShareableLinkDialog,
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
<style scoped>
button {
  border: thin solid rgba(0, 0, 0, 0.12);
}
</style>
