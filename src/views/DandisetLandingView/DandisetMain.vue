<template>
  <div>
    <v-card
      outlined
    >
      <v-row class="mx-2 my-2">
        <v-col>
          <h1 class="font-weight-light">
            {{ meta.name }}
          </h1>
        </v-col>
      </v-row>
      <v-row class="mx-1">
        <v-col cols="3">
          <v-chip
            outlined
          >
            <v-row>
              <v-col cols="6">
                <span class="pr-3">ID: {{ currentDandiset.dandiset.identifier }}</span>
              </v-col>
              <v-col
                cols="6"
                class="orange lighten-4"
              >
                <span class="orange--text text--darken-4">
                  {{ currentDandiset.version.toUpperCase() }}
                </span>
              </v-col>
            </v-row>
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
        >
          <v-card
            outlined
          >
            <v-card-text
              v-if="meta.keywords"
              style="border-bottom: thin solid rgba(0, 0, 0, 0.12);"
            >
              Keywords:
              <v-chip
                v-for="keyword in meta.keywords"
                :key="keyword"
                small
                style="margin: 5px;"
              >
                {{ keyword }}
              </v-chip>
            </v-card-text>
            <v-card-text>
              Licenses:
              <v-chip
                v-for="license in meta.license"
                :key="license"
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
  defineComponent, ref, computed, ComputedRef,
} from '@vue/composition-api';

import moment from 'moment';

import {
  loggedIn as loggedInFunc, user as userFunc,
} from '@/rest';

import { User, Version } from '@/types';

import AccessInformationTab from '@/components/DLP/AccessInformationTab.vue';
import AssetSummaryTab from '@/components/DLP/AssetSummaryTab.vue';
import ContributorsTab from '@/components/DLP/ContributorsTab.vue';
import OverviewTab from '@/components/DLP/OverviewTab.vue';
import RelatedResourcesTab from '@/components/DLP/RelatedResourcesTab.vue';
import SubjectMatterTab from '@/components/DLP/SubjectMatterTab.vue';

const tabs = [
  {
    name: 'Overview',
    href: 'overview',
    component: OverviewTab,
  },
  {
    name: 'Contributors',
    href: 'contributors',
    component: ContributorsTab,
    icon: 'mdi-account',
  },
  {
    name: 'Subject Matter',
    href: 'subject-matter',
    component: SubjectMatterTab,
    icon: 'mdi-notebook-outline',
  },
  {
    name: 'Access Information',
    href: 'access-information',
    component: AccessInformationTab,
    icon: 'mdi-account-question',
  },
  {
    name: 'Asset Summary',
    href: 'asset-summary',
    component: AssetSummaryTab,
    icon: 'mdi-clipboard-list',
  },
  {
    name: 'Related Resources',
    href: 'related-resources',
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
    userCanModifyDandiset: {
      type: Boolean,
      required: true,
    },
  },
  setup(props, ctx) {
    const titleClasses = 'mx-1';
    const mainFields = ['name', 'description', 'identifier'];

    const store = ctx.root.$store;

    const currentDandiset: ComputedRef<Version> = computed(
      () => store.state.dandiset.publishDandiset,
    );
    const versions: ComputedRef<Version[]> = computed(
      () => store.state.dandiset.versions,
    );
    const user: ComputedRef<User|null> = computed(userFunc);
    const loggedIn: ComputedRef<boolean> = computed(loggedInFunc);

    const currentTab = ref(null);

    function formatDate(date: string) {
      return moment(date).format('LL');
    }

    return {
      titleClasses,
      mainFields,
      currentDandiset,
      versions,
      user,
      loggedIn,
      formatDate,
      currentTab,
      tabs,
    };
  },
});
</script>
