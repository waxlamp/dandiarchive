<template>
  <div>
    <v-card class="pb-2">
      <v-row
        class="mx-2 font-weight-regular"
        align="center"
      >
        <v-col>
          Dandiset ID: {{ meta.id }}
        </v-col>
      </v-row>
      <v-row class="mx-2 my-2">
        <v-col>
          <h1 class="font-weight-regular">
            {{ meta.name }}
          </h1>
        </v-col>
      </v-row>
      <v-row
        class="mx-2"
        align="center"
      >
        <v-col>
          Share
          <v-dialog
            v-model="dialog"
            offset-y
            min-width="420"
            max-width="500"
          >
            <template #activator="{ on }">
              <v-icon
                color="primary"
                v-on="on"
              >
                mdi-share-variant
              </v-icon>
            </template>

            <v-card>
              <v-card-title>
                <v-btn
                  icon
                  x-small
                  :right="true"
                  :absolute="true"
                  @click="dialog = false"
                >
                  <v-icon>mdi-close</v-icon>
                </v-btn>
              </v-card-title>
              <v-card-title class="text-h6">
                {{ meta.name }}
              </v-card-title>
              <v-card-text>
                <div>
                  <span
                    v-for="author in contributors"
                    :key="author.name + author.identifier"
                    class="text-body-1"
                  >
                    {{ author.name }}
                  </span>
                </div>
              </v-card-text>
              <v-card-text>
                <span class="font-weight-black">
                  Share this article:
                </span>
                <CopyText
                  class="pa-2"
                  :text="permalink"
                  icon-hover-text="Copy permalink to clipboard"
                />
              </v-card-text>
              <v-divider class="mx-4" />
              <v-card-actions>
                <v-list-item class="grow">
                  <v-row
                    align="center"
                  >
                    <ShareNetwork
                      network="twitter"
                      :url="permalink"
                      :title="meta.name"
                      :hashtags="hashtags"
                    >
                      <v-icon
                        class="mr-1"
                        color="blue"
                        large
                      >
                        mdi-twitter
                      </v-icon>
                    </ShareNetwork>
                  </v-row>
                </v-list-item>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-col>
        <v-col v-if="meta.citation">
          Cite as
          <v-menu
            offset-y
            :close-on-content-click="false"
            min-width="420"
            max-width="420"
          >
            <template #activator="{ on }">
              <v-icon
                color="primary"
                v-on="on"
              >
                mdi-comment-quote-outline
              </v-icon>
            </template>

            <v-card>
              <CopyText
                class="pa-2"
                :text="meta.citation"
                :is-textarea="true"
                icon-hover-text="Copy to clipboard"
              />
            </v-card>
          </v-menu>
        </v-col>
        <DownloadDialog>
          <template #activator="{ on }">
            <v-btn
              id="download"
              text
              v-on="on"
            >
              <v-icon
                color="primary"
                class="mr-2"
              >
                mdi-download
              </v-icon>
              Download
              <v-icon>mdi-menu-down</v-icon>
            </v-btn>
          </template>
        </DownloadDialog>
        <v-btn
          id="view-data"
          :to="fileBrowserLink"
          text
        >
          <v-icon
            color="primary"
            class="mr-2"
          >
            mdi-file-tree
          </v-icon>
          View Data
        </v-btn>
        <v-btn
          id="view-edit-metadata"
          text
          @click="$emit('edit')"
        >
          <v-icon
            color="primary"
            class="mr-2"
          >
            {{ metadataButtonIcon }}
          </v-icon>
          {{ metadataButtonText }}
        </v-btn>
        <template v-if="publishDandiset.version === 'draft'">
          <v-tooltip
            left
            :disabled="!publishDisabledMessage"
          >
            <template #activator="{ on }">
              <div v-on="on">
                <v-btn
                  id="publish"
                  text
                  :disabled="publishDisabled || !user"
                  @click="publish"
                >
                  <v-icon
                    color="success"
                    class="mr-2"
                  >
                    mdi-publish
                  </v-icon>
                  Publish
                </v-btn>
              </div>
            </template>
            <p
              :style="{ whiteSpace: 'pre-line' }"
              v-text="publishDisabledMessage"
            />
          </v-tooltip>
          <v-menu
            v-if="publishDandiset.version_validation_errors.length"
            :nudge-width="200"
            offset-y
          >
            <template #activator="{ on: menu, attrs }">
              <v-tooltip bottom>
                <template #activator="{ on: tooltip }">
                  <v-btn
                    v-bind="attrs"
                    v-on="{ ...tooltip, ...menu }"
                  >
                    <v-icon
                      color="error"
                      class="mr-1"
                    >
                      mdi-clipboard-alert
                    </v-icon>
                    {{ publishDandiset.version_validation_errors.length }}
                  </v-btn>
                </template>
                <span>Fix issues with metadata</span>
              </v-tooltip>
            </template>
            <v-card class="pa-1">
              <v-list
                style="max-height: 200px"
                class="overflow-y-auto"
              >
                <div
                  v-for="(error, index) in publishDandiset.version_validation_errors"
                  :key="index"
                >
                  <v-list-item>
                    <v-list-item-icon>
                      <v-icon>
                        {{ getValidationErrorIcon(error.field) }}
                      </v-icon>
                    </v-list-item-icon>

                    <v-list-item-content>
                      {{ error.field }}: {{ error.message }}
                    </v-list-item-content>
                  </v-list-item>
                  <v-divider />
                </div>
              </v-list>
              <v-btn
                color="primary"
                @click="$emit('edit')"
              >
                Fix issues
              </v-btn>
            </v-card>
          </v-menu>
          <v-menu
            v-if="publishDandiset.asset_validation_errors.length"
            :nudge-width="300"
            offset-y
          >
            <template #activator="{ on: menu, attrs }">
              <v-tooltip bottom>
                <template #activator="{ on: tooltip }">
                  <v-btn
                    v-bind="attrs"
                    v-on="{ ...tooltip, ...menu }"
                  >
                    <v-icon
                      color="error"
                      class="mr-1"
                    >
                      mdi-database-alert
                    </v-icon>
                    {{ publishDandiset.asset_validation_errors.length }}
                  </v-btn>
                </template>
                <span>Fix issues with assets</span>
              </v-tooltip>
            </template>
            <v-card class="pa-1">
              <v-list
                style="max-height: 200px"
                class="overflow-y-auto"
              >
                <div
                  v-for="(error, index) in publishDandiset.asset_validation_errors"
                  :key="index"
                >
                  <v-list-item>
                    <v-list-item-icon>
                      <v-icon>
                        {{ getValidationErrorIcon(error.field) }}
                      </v-icon>
                    </v-list-item-icon>

                    <v-list-item-content>
                      {{ error.field }}: {{ error.message }}
                    </v-list-item-content>
                  </v-list-item>
                  <v-divider />
                </div>
              </v-list>
            </v-card>
          </v-menu>
        </template>
      </v-row>

      <v-divider />
      <v-row
        v-if="contributors.length"
        class="mx-2"
        align="center"
      >
        <v-col>
          <span
            v-for="author in contributors"
            :key="author.name + author.identifier"
          >
            <a
              v-if="author.identifier"
              :href="author.identifier"
              target="_blank"
            >
              <img
                alt="ORCID logo"
                src="https://info.orcid.org/wp-content/uploads/2019/11/orcid_16x16.png"
                width="16"
                height="16"
              ></a>
            <v-tooltip
              v-if="author.affiliation"
              top
              color="black"
            >
              <template #activator="{ on }">
                <span v-on="on">
                  {{ author.name }}
                </span>
              </template>
              <span>{{ author.affiliation }}</span>
            </v-tooltip>
            <span v-else> {{ author.name }}</span>
          </span>
        </v-col>
      </v-row>
      <v-row
        v-if="meta.keywords"
        class="mx-2"
      >
        <v-col>
          <span
            v-for="key in meta.keywords"
            :key="key"
          >
            <v-chip
              small
              style="margin: 5px;"
              class="grey darken-2 font-weight-bold white--text"
            > {{ key }} </v-chip>
          </span>
        </v-col>
      </v-row>
      <v-row :class="titleClasses">
        <v-card-title class="font-weight-regular">
          Description
        </v-card-title>
      </v-row>
      <v-row class="mx-1 mb-4 px-4 font-weight-light">
        {{ meta.description }}
      </v-row>
      <template v-for="key in Object.keys(extraFields).sort()">
        <template v-if="renderData(extraFields[key], schema.properties[key])">
          <v-divider :key="`${key}-divider`" />
          <v-row
            :key="`${key}-title`"
            :class="titleClasses"
          >
            <v-card-title class="font-weight-regular">
              {{ schemaPropertiesCopy[key].title || key }}
            </v-card-title>
          </v-row>
          <v-row
            :key="key"
            class="mx-2 mb-4"
          >
            <v-col class="py-0">
              <ListingComponent
                :field="key"
                :schema="schema.properties[key]"
                :data="extraFields[key]"
              />
            </v-col>
          </v-row>
        </template>
      </template>
    </v-card>
  </div>
</template>

<script>
import { mapState, mapGetters } from 'vuex';

import { dandiUrl, VALIDATION_ICONS } from '@/utils/constants';
import {
  loggedIn, publishRest, user,
} from '@/rest';

import CopyText from '@/components/CopyText.vue';
import _ from 'lodash';
import DownloadDialog from './DownloadDialog.vue';
import ListingComponent from './ListingComponent.vue';

export default {
  name: 'DandisetMain',
  components: {
    CopyText,
    DownloadDialog,
    ListingComponent,
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
  data() {
    return {
      dialog: false,
      titleClasses: 'mx-1',
      mainFields: [
        'name',
        'description',
        'identifier',
      ],
      hashtags: 'dandi',
    };
  },
  computed: {
    loggedIn,
    user,
    contributors() {
      // eslint-disable-next-line no-console
      const persons = _.filter(this.meta.contributor, (author) => author.schemaKey === 'Person' && author.includeInCitation);
      const authors = _.map(persons, (author, index) => {
        let affiliations = '';
        let orcid_id = author.identifier;
        if (!_.isEmpty(author.affiliation)) {
          affiliations = _.map(author.affiliation, (a) => a.name);
          affiliations = affiliations.join(', ');
        }
        let author_name = author.name;
        if (index < persons.length - 1) {
          author_name = `${author.name};`;
        }
        if (orcid_id) {
          orcid_id = `https://orcid.org/${orcid_id}`;
        }
        return { name: author_name, identifier: orcid_id, affiliation: affiliations };
      });
      return authors;
    },
    publishDisabledMessage() {
      if (!this.loggedIn) {
        return 'You must be logged in to edit.';
      }
      if (!this.userCanModifyDandiset) {
        return 'You do not have permission to edit this dandiset.';
      }
      if (this.publishDandiset.status === 'Pending') {
        return 'This dandiset has not yet been validated.';
      }
      if (this.publishDandiset.status === 'Validating') {
        return 'Currently validating this dandiset.';
      }
      if (this.publishDandiset.status === 'Published') {
        return 'No changes since last publish.';
      }

      return null;
    },
    publishDisabled() {
      return !!(this.publishDandiset.version_validation_errors.length
        || this.publishDandiset.asset_validation_errors.length
        || this.publishDisabledMessage);
    },
    metadataButtonText() {
      return this.userCanModifyDandiset ? 'Edit metadata' : 'View metadata';
    },
    metadataButtonIcon() {
      return this.userCanModifyDandiset ? 'mdi-pencil' : 'mdi-eye';
    },
    fileBrowserLink() {
      const { version } = this;
      const { identifier } = this.publishDandiset.dandiset;
      // TODO: this probably does not work correctly yet
      return {
        name: 'fileBrowser',
        params: { identifier, version },
        query: {
          location: '',
        },
      };
    },
    permalink() {
      return `${dandiUrl}/dandiset/${this.publishDandiset.dandiset.identifier}/${this.version}`;
    },
    extraFields() {
      const { meta, mainFields } = this;
      let extra = Object.keys(meta).filter(
        (x) => !mainFields.includes(x) && x in this.schema.properties,
      );
      const remove_list = ['citation', 'repository', 'url', 'schemaVersion', 'version', 'id', 'keywords', 'schemaKey'];
      extra = extra.filter((n) => !remove_list.includes(n));
      const extra_obj = extra.reduce((obj, key) => ({ ...obj, [key]: meta[key] }), {});
      extra_obj.contributor = _.filter(meta.contributor, (author) => author.schemaKey !== 'Person');
      delete extra_obj.assetsSummary.schemaKey;
      delete extra_obj.assetsSummary.numberOfBytes;
      delete extra_obj.assetsSummary.numberOfFiles;
      _.forEach(extra_obj, (val) => {
        if (Array.isArray(val) && val.length !== 0) {
          val.forEach((each_val) => {
            if (Object.prototype.hasOwnProperty.call(each_val, 'schemaKey')) {
              /* eslint no-param-reassign:["error",
              {"ignorePropertyModificationsFor":["each_val"] }] */
              delete each_val.schemaKey;
            }
          });
        }
      });
      return extra_obj;
    },
    schemaPropertiesCopy() {
      const schema_copy = JSON.parse(JSON.stringify(this.schema.properties));
      schema_copy.contributor.title = 'Funding Information';
      return schema_copy;
    },
    ...mapState('dandiset', {
      publishDandiset: (state) => state.publishDandiset,
    }),
    ...mapGetters('dandiset', ['version']),
  },
  methods: {
    async publish() {
      const version = await publishRest.publish(this.publishDandiset.dandiset.identifier);
      // re-initialize the dataset to load the newly published version
      await this.$store.dispatch('dandiset/initializeDandisets', { identifier: version.dandiset.identifier, version: version.version });
    },
    renderData(data, schema) {
      if (data === null || _.isEmpty(data)) {
        return false;
      }
      if (schema.type === 'array' && Array.isArray(data) && data.length === 0) {
        return false;
      }
      return true;
    },
    getValidationErrorIcon(errorField) {
      const icons = Object.keys(VALIDATION_ICONS).filter((field) => errorField.includes(field));
      if (icons.length > 0) {
        return VALIDATION_ICONS[icons[0]];
      }
      return VALIDATION_ICONS.DEFAULT;
    },
  },
};
</script>
