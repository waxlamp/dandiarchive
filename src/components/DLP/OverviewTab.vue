<template>
  <div>
    <v-card>
      <v-card-title>
        <v-icon>mdi-account-multiple</v-icon>
        <span class="ml-2">Contributors</span>
      </v-card-title>
      <v-card-text>
        <v-chip
          v-for="(contributor, i) in contributors"
          :key="i"
          style="margin: 5px;"
        >
          {{ contributor.name }}
          <a
            v-if="contributor.identifier"
            :href="`https://orcid.org/${contributor.identifier}`"
            target="_blank"
            class="ml-1"
          >
            <img
              alt="ORCID logo"
              src="https://info.orcid.org/wp-content/uploads/2019/11/orcid_16x16.png"
              width="16"
              height="16"
            ></a>
        </v-chip>
      </v-card-text>
    </v-card>
  </div>
</template>

<script lang="ts">
import { computed, defineComponent } from '@vue/composition-api';

export default defineComponent({
  name: 'OverviewTab',
  components: {
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
    const { meta } = props;

    const contributors = computed(
      () => meta.contributor.filter(
        (author: any) => author.schemaKey === 'Person' && author.includeInCitation,
      ),
    );

    return {
      contributors,
    };
  },
});
</script>
