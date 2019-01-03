<template>
  <v-layout row>
    <v-snackbar v-model="showOperationStatus" left>{{operationStatus}}</v-snackbar>
    <v-flex xs12 sm6 offset-sm3>
      <v-card>
        <v-toolbar color="cyan" dark>
          <v-toolbar-side-icon></v-toolbar-side-icon>

          <v-toolbar-title>Person List</v-toolbar-title>

          <v-spacer></v-spacer>

          <v-btn icon @click="loadCsv">
            <v-icon>file_upload</v-icon>
          </v-btn>
        </v-toolbar>

        <v-list three-line>
          <v-subheader>{{persons.length}} records in person database</v-subheader>
          <template v-for="(person, index) in persons">
            <v-divider :inset="true" :key="index"></v-divider>

            <v-list-tile :key="'person'+index" avatar>
              <v-list-tile-avatar>
                <img
                  :src="person.avatar"
                >
              </v-list-tile-avatar>

              <v-list-tile-content>
                <v-list-tile-title v-html="`${person.first_name} ${person.last_name}`"></v-list-tile-title>
                <v-list-tile-sub-title v-html="`Email: ${person.email}`"></v-list-tile-sub-title>
                <v-list-tile-sub-title v-html="`IP: ${person.ip_address}`"></v-list-tile-sub-title>
              </v-list-tile-content>
              <v-list-tile-action>
                <v-btn icon ripple @click="deletePerson(person)">
                  <v-icon color="grey lighten-1">delete</v-icon>
                </v-btn>
              </v-list-tile-action>
            </v-list-tile>
          </template>
        </v-list>
      </v-card>
    </v-flex>
  </v-layout>
</template>

<script>
const axios = require("axios");
import { nSQL } from "nano-sql";

export default {
  name: "PersonList",
  data() {
    return {
      showOperationStatus: false,
      operationStatus: "",
      persons: [],
      updateObservable: null
    };
  },
  created: function() {
    nSQL().onConnected(() => {
      this.subscribeDbUpdates();
    });
  },
  mounted() {
    document.addEventListener(
      typeof cordova !== "undefined" ? "deviceready" : "DOMContentLoaded",
      () => {
        let model = [
          { key: "id", type: "int", props: ["pk", "ai"] },
          { key: "first_name", type: "string" },
          { key: "last_name", type: "string" },
          { key: "email", type: "string" },
          { key: "gender", type: "string" },
          { key: "ip_address", type: "string" },
          { key: "avatar", type: "string" }
        ];
        if (window.nSQLite && window.cordova.platformId != "browser") {
          nSQL("persondb")
            .model(model)
            .config({
              mode: window.nSQLite.getMode() // required
            })
            .connect();
        } else {
          nSQL("persondb")
            .model(model)
            .connect();
        }
      }
    );
  },
  methods: {
    subscribeDbUpdates() {
      this.updateObservable = nSQL()
        .observable(() =>
          nSQL("persondb")
            .query("select")
            .emit()
        )
        .debounce(100)
        .filter((rows, idx) => rows.length > 0)
        .subscribe((rows, event) => {
          console.log(rows.length)
          this.persons = rows;
        });
    },
    unsubscribeDbUpdates() {
      this.updateObservable.unsubscribe();
    },
    loadCsv() {
      axios.get("MOCK_DATA.csv", {}).then(response => {
        nSQL()
          .loadCSV("persondb", response.data)
          .then(rows => {
            this.showOperationStatus = true;
            this.operationStatus = `${rows.length} rows added`;
          });
      });
    },
    deletePerson(person) {
      nSQL("persondb")
        .query("delete")
        .where(["id", "=", person.id])
        .exec()
        .then(rows => {
          this.showOperationStatus = true;
          this.operationStatus = `${rows.length} rows deleted`;
        });
    }
  }
};
</script>


<style scoped>
</style>
