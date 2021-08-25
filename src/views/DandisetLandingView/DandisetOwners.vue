<template>
  <div>
    <template v-if="!owners || !owners.length">
      <v-row
        align="center"
        justify="center"
        class="mx-1"
      >
        No owners
      </v-row>
    </template>
    <template v-else>
      <v-row class="mx-1 px-1">
        <v-autocomplete
          v-model="selection"
          :items="items"
          :loading="loadingUsers"
          :search-input.sync="search"
          hide-no-data
          clearable
          auto-select-first
          item-text="result"
          placeholder="enter email address"
          outlined
          flat
          return-object
          @update:search-input="throttledUpdate"
        />
      </v-row>
      <v-row class="mx-1">
        <v-chip
          v-for="(owner, i) in newOwners"
          :key="i"
          color="light-blue lighten-4"
          text-color="light-blue darken-3"
          class="font-weight-medium ma-1"
          :close="userCanModifyDandiset"
          @click:close="removeOwner(i)"
        >
          {{ owner.name || owner.username }}
        </v-chip>
      </v-row>
    </template>
  </div>
</template>

<script>
import { publishRest, user } from '@/rest';
import { mapState, mapMutations } from 'vuex';
import _ from 'lodash';

// Includes a field `result` on each user which is the value displayed in the UI
const appendResult = (users) => users.map((u) => ({ ...u, result: (u.name) ? `${u.name} (${u.username})` : u.username }));

export default {
  name: 'DandisetOwners',
  props: {
    owners: {
      type: Array,
      required: true,
    },
  },
  data() {
    return {
      search: null,
      loadingUsers: false,
      selection: null,
      newOwners: appendResult(this.owners),
      items: [],
      throttledUpdate: _.debounce(this.updateItems, 200),
    };
  },
  asyncComputed: {
    async userCanModifyDandiset() {
      if (this.publishDandiset?.metadata?.version !== 'draft' || !user()) {
        return false;
      }
      if (user.value?.admin) {
        return true;
      }
      const { data: owners } = await publishRest.owners(this.publishDandiset.dandiset.identifier);
      const userExists = owners.find((owner) => owner.username === user().username);
      return !!userExists;
    },
  },
  computed: {
    user,
    ...mapState('dandiset', ['publishDandiset']),
  },
  watch: {
    async selection(val) {
      // Verify that the selected user hasn't already been selected
      if (val && !this.newOwners.find((x) => x.username === val.username)) {
        this.newOwners.push(val);
      }
      // Clear the search field, if it isn't already
      if (val) {
        this.selection = '';
      }

      await this.save();
    },
  },
  methods: {
    async updateItems() {
      if (!this.search) return;

      this.loadingUsers = true;
      const users = await publishRest.searchUsers(this.search);
      this.items = appendResult(users);
      this.loadingUsers = false;
    },
    async removeOwner(index) {
      this.newOwners.splice(index, 1);
      await this.save();
    },
    async save() {
      const { identifier } = this.publishDandiset.dandiset;
      const { data } = await publishRest.setOwners(identifier, this.newOwners);
      this.setOwners(data);
    },
    ...mapMutations('dandiset', ['setOwners']),
  },
};
</script>
