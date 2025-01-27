<template>
  <v-card class="flex-grow-0">
    <v-card-title>Manage Ownership</v-card-title>
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
      <v-row class="mx-1 px-6">
        <v-autocomplete
          v-model="selection"
          :items="items"
          :loading="loadingUsers"
          :search-input.sync="search"
          hide-no-data
          clearable
          auto-select-first
          item-text="result"
          placeholder="Search by first name, last name or username"
          outlined
          flat
          return-object
          @update:search-input="throttledUpdate"
        />
      </v-row>
      <v-row class="mx-1">
        <v-list
          width="100%"
          style="overflow-y: auto;"
          class="px-6"
          max-height="50vh"
        >
          <template v-for="(owner, i) in newOwners">
            <v-list-item :key="owner.username">
              <v-list-item-title>
                {{ owner.result }}
              </v-list-item-title>
              <v-list-item-action>
                <v-btn
                  icon
                  @click="removeOwner(i)"
                >
                  <v-icon>mdi-close</v-icon>
                </v-btn>
              </v-list-item-action>
            </v-list-item>
            <v-divider :key="`${owner.username}-divider`" />
          </template>
        </v-list>
      </v-row>
    </template>
    <v-row
      justify="end"
      align="end"
      class="mx-1 pt-4 pb-2"
    >
      <v-btn
        tile
        text
        @click="close"
      >
        Cancel
      </v-btn>
      <v-btn
        tile
        text
        color="primary"
        @click="save"
      >
        Save Changes
      </v-btn>
    </v-row>
  </v-card>
</template>

<script>
import { publishRest, user } from '@/rest';
import { mapState, mapMutations } from 'vuex';
import _ from 'lodash';

// Includes a field `result` on each user which is the value displayed in the UI
const appendResult = (users) => users.map((u) => ({ ...u, result: (u.name) ? `${u.name} (${u.username})` : u.username }));

export default {
  name: 'DandisetOwnersDialog',
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
  computed: {
    user,
    ...mapState('dandiset', ['publishDandiset']),
  },
  watch: {
    selection(val) {
      // Verify that the selected user hasn't already been selected
      if (val && !this.newOwners.find((x) => x.username === val.username)) {
        this.newOwners.push(val);
      }
      // Clear the search field, if it isn't already
      if (val) {
        this.selection = '';
      }
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
    removeOwner(index) {
      this.newOwners.splice(index, 1);
    },
    close() {
      this.$emit('close');
    },
    async save() {
      const { identifier } = this.publishDandiset.dandiset;
      const { data } = await publishRest.setOwners(identifier, this.newOwners);
      this.setOwners(data);
      this.close();
    },
    ...mapMutations('dandiset', ['setOwners']),
  },
};
</script>
