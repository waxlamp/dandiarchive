<template>
  <v-progress-linear
    v-if="!currentDandiset"
    indeterminate
  />
  <PublishFileBrowser
    v-else
    :identifier="identifier"
    :version="version"
  />
</template>

<script>
import { draftVersion } from '@/utils/constants';
import PublishFileBrowser from './PublishFileBrowser.vue';

export default {
  name: 'FileBrowser',
  components: {
    PublishFileBrowser,
  },
  props: {
    identifier: {
      type: String,
      required: true,
    },
    version: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
      draftVersion,
    };
  },
  computed: {
    currentDandiset() {
      return this.$store.direct.state.dandiset.dandiset;
    },
  },
  async created() {
    // Don't extract currentDandiset, for reactivity
    const { identifier, version } = this;
    if (!this.currentDandiset) {
      this.fetchDandiset({ identifier, version });
    }
  },
  methods: {
    fetchDandiset() {
      this.$store.direct.dispatch.fetchDandiset({
        identifier: this.identifier,
        version: this.version,
      });
    },
  },
};
</script>
