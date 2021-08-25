<template>
  <div>
    <v-card
      class="px-3"
      outlined
    >
      <v-row class="mx-2 my-2 mb-0">
        <v-col>
          <h1 class="font-weight-light">
            {{ meta.name }}
          </h1>
        </v-col>
      </v-row>
      <v-row class="mx-1">
        <v-col cols="3">
          <v-chip
            class="text-wrap py-1"
            style="text-align: center;"
            outlined
          >
            <span>
              ID: <span class="font-weight-bold">{{ currentDandiset.dandiset.identifier }}</span>
            </span>
            <v-divider
              vertical
              class="mx-2"
            />
            <span
              :class="`
                font-weight-bold
                ${currentDandiset.version === 'draft' ? 'orange' : 'blue'}--text text--darken-4
              `"
            >
              {{ currentDandiset.version.toUpperCase() }}
            </span>
          </v-chip>
        </v-col>
        <v-col cols="3">
          <span>
            <v-icon class="grey--text text--lighten-1">mdi-account</v-icon>
            Contact <strong>{{ currentDandiset.contact_person }}</strong>
          </span>
        </v-col>
        <v-col cols="3">
          <span>
            <v-icon class="grey--text text--lighten-1">mdi-calendar-range</v-icon>
            Created <strong>{{ formatDate(currentDandiset.created) }}</strong>
          </span>
        </v-col>
        <v-col cols="3">
          <span>
            <v-icon class="grey--text text--lighten-1">mdi-history</v-icon>
            Last update <strong>{{ formatDate(currentDandiset.modified) }}</strong>
          </span>
        </v-col>
      </v-row>

      <v-divider />

      <v-row class="mx-1 my-4 px-4 font-weight-light">
        {{ meta.description }}
      </v-row>

      <v-row class="justify-center">
        <v-col
          cols="11"
          class="pb-0"
        >
          <v-card
            v-if="(meta.keywords && meta.keywords.length) || (meta.license && meta.license.length)"
            outlined
            class="mb-4"
          >
            <v-card-text
              v-if="meta.keywords.length"
              style="border-bottom: thin solid rgba(0, 0, 0, 0.12);"
            >
              Keywords:
              <v-chip
                v-for="(keyword, i) in meta.keywords"
                :key="i"
                small
                style="margin: 5px;"
              >
                {{ keyword }}
              </v-chip>
            </v-card-text>

            <v-card-text v-if="meta.license.length">
              Licenses:
              <v-chip
                v-for="(license, i) in meta.license"
                :key="i"
                small
                style="margin: 5px;"
              >
                {{ license }}
              </v-chip>
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>

      <v-tabs
        v-model="currentTab"
        class="ml-3"
        show-arrows
      >
        <v-tabs-slider />

        <v-tab
          v-for="(tab, index) in tabs"
          :key="tab.name"
          :href="`#${index}`"
        >
          <v-icon>{{ tab.icon }}</v-icon>
          {{ tab.name }}
        </v-tab>
      </v-tabs>
    </v-card>

    <!-- Dynamically render component based on current tab -->
    <v-row class="justify-center">
      <v-col cols="11">
        <component
          :is="tabs[currentTab].component"
          v-if="tabs[currentTab]"
          v-bind="{ schema, meta }"
        />
      </v-col>
    </v-row>
  </div>
</template>

<script lang="ts">
import {
  defineComponent, ref, computed, ComputedRef, Ref,
} from '@vue/composition-api';

import moment from 'moment';

import { Version } from '@/types';

import AccessInformationTab from '@/components/DLP/AccessInformationTab.vue';
import AssetSummaryTab from '@/components/DLP/AssetSummaryTab.vue';
import ContributorsTab from '@/components/DLP/ContributorsTab.vue';
import OverviewTab from '@/components/DLP/OverviewTab.vue';
import RelatedResourcesTab from '@/components/DLP/RelatedResourcesTab.vue';
import SubjectMatterTab from '@/components/DLP/SubjectMatterTab.vue';

const tabs = [
  {
    name: 'Overview',
    component: OverviewTab,
  },
  {
    name: 'Contributors',
    component: ContributorsTab,
    icon: 'mdi-account',
  },
  {
    name: 'Subject Matter',
    component: SubjectMatterTab,
    icon: 'mdi-notebook-outline',
  },
  {
    name: 'Access Information',
    component: AccessInformationTab,
    icon: 'mdi-account-question',
  },
  {
    name: 'Asset Summary',
    component: AssetSummaryTab,
    icon: 'mdi-clipboard-list',
  },
  {
    name: 'Related Resources',
    component: RelatedResourcesTab,
    icon: 'mdi-book',
  },
];

export default defineComponent({
  name: 'DandisetMain',
  components: {
    AccessInformationTab,
    AssetSummaryTab,
    ContributorsTab,
    OverviewTab,
    RelatedResourcesTab,
    SubjectMatterTab,
  },
  props: {
    schema: {
      type: Object,
      required: true,
    },
    meta: {
      type: Object,
      required: true,
    },
  },
  setup(props, ctx) {
    const store = ctx.root.$store;

    const currentDandiset: ComputedRef<Version> = computed(
      () => store.state.dandiset.publishDandiset,
    );

    const currentTab: Ref<string> = ref('');

    function formatDate(date: string): string {
      return moment(date).format('LL');
    }

    return {
      currentDandiset,
      formatDate,
      currentTab,
      tabs,
    };
  },
});
</script>
