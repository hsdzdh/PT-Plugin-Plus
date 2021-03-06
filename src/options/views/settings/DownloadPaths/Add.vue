<template>
  <div>
    <v-dialog v-model="show" max-width="800">
      <v-card>
        <v-card-title
          class="headline blue-grey darken-2"
          style="color:white"
        >{{ $t('settings.downloadPaths.add.title') }}</v-card-title>

        <v-card-text>
          <v-form v-model="valid">
            <v-autocomplete
              v-model="selectedSite"
              :items="getSites()"
              :label="$t('settings.downloadPaths.add.selectSite')"
              persistent-hint
              single-line
              item-text="name"
              item-value="host"
              return-object
              :rules="rules.require"
            >
              <template slot="selection" slot-scope="{ item }">
                <v-list-tile-avatar>
                  <img :src="item.icon">
                </v-list-tile-avatar>
                <span v-text="item.name"></span>
              </template>
              <template slot="item" slot-scope="data" style>
                <v-list-tile-avatar>
                  <img :src="data.item.icon">
                </v-list-tile-avatar>
                <v-list-tile-content>
                  <v-list-tile-title v-html="data.item.name"></v-list-tile-title>
                  <v-list-tile-sub-title v-html="data.item.url"></v-list-tile-sub-title>
                </v-list-tile-content>
                <v-list-tile-action>
                  <v-list-tile-action-text>{{ joinTags(data.item.tags) }}</v-list-tile-action-text>
                </v-list-tile-action>
              </template>
            </v-autocomplete>
            <v-textarea
              v-model="paths"
              :label="$t('settings.downloadPaths.add.path')"
              value
              :hint="$t('settings.downloadPaths.add.pathTip')"
              :rules="rules.require"
            ></v-textarea>
            <v-alert :value="true" color="info" icon="info" outline v-if="client.pathDescription">
              <div v-html="client.pathDescription"></div>
              <KeyDescription/>
            </v-alert>
          </v-form>
        </v-card-text>

        <v-divider></v-divider>

        <v-card-actions class="pa-3">
          <v-spacer></v-spacer>
          <v-btn flat color="error" @click="cancel">
            <v-icon>cancel</v-icon>
            <span class="ml-1">{{ $t('settings.downloadPaths.add.cancel') }}</span>
          </v-btn>
          <v-btn flat color="success" @click="save" :disabled="!valid">
            <v-icon>check_circle_outline</v-icon>
            <span class="ml-1">{{ $t('settings.downloadPaths.add.ok') }}</span>
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>
<script lang="ts">
import { Site, DownloadClient } from "@/interface/common";
import Vue from "vue";
import KeyDescription from "./KeyDescription.vue";

export default Vue.extend({
  components: {
    KeyDescription
  },
  data() {
    return {
      rules: {
        require: [(v: any) => !!v || "!"]
      },
      show: false,
      valid: false,
      paths: "",
      selectedSite: {}
    };
  },
  props: {
    value: Boolean,
    client: Object
  },
  model: {
    prop: "value",
    event: "change"
  },
  watch: {
    show() {
      this.$emit("change", this.show);
      if (!this.show) {
        this.paths = "";
        this.selectedSite = {};
      }
    },
    value() {
      this.show = this.value;
    }
  },
  methods: {
    save() {
      this.$emit("save", {
        site: this.selectedSite,
        paths: this.paths.split("\n")
      });
      this.show = false;
    },
    cancel() {
      this.show = false;
    },
    joinTags(tags: any): string {
      if (tags && tags.join) {
        return tags.join(", ");
      }
      return "";
    },
    getSites(): Site[] {
      let clients = this.$store.state.options.clients;
      let sites = this.$store.state.options.sites;
      let result: Site[] = [];
      if (clients && clients.length) {
        let client: DownloadClient = clients.find((item: DownloadClient) => {
          return item.id === this.client.id;
        });
        if (client && client.paths) {
          sites.forEach((site: Site) => {
            if (!client.paths.hasOwnProperty(site.host)) {
              result.push(site);
            }
          });
        } else {
          result = sites;
        }
        return result;
      }
      return sites;
    }
  },
  computed: {},
  created() {}
});
</script>
