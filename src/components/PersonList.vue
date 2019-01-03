<template>
  <v-layout row>
    <v-flex xs12 sm6 offset-sm3>
      <v-card>
        <v-toolbar color="cyan" dark>
          <v-toolbar-side-icon></v-toolbar-side-icon>

          <v-toolbar-title>Person List</v-toolbar-title>

          <v-spacer></v-spacer>

          <v-btn icon>
            <v-icon>search</v-icon>
          </v-btn>
        </v-toolbar>

        <v-list three-line>
          <v-subheader>{{persons.length}} records in person database</v-subheader>
          <template v-for="(person, index) in persons">
            <v-divider inset="true" :key="index"></v-divider>

            <v-list-tile :key="'person'+index" avatar>
              <v-list-tile-avatar>
                <img
                  :src="'https://picsum.photos/200/300/?random&r=' + Math.round(Math.random() * 1000)"
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
      persons: []
    };
  },
  mounted() {
    nSQL().onConnected(() => {
      axios.get("MOCK_DATA.csv", {}).then(response => {
        nSQL()
          .loadCSV("persondb", response.data)
          .then(() => {
            nSQL("persondb")
              .query("select")
              .exec()
              .then(rows => {
                this.persons = rows;
              });
          });
      });
    });

    document.addEventListener(
      typeof cordova !== "undefined" ? "deviceready" : "DOMContentLoaded",
      () => {
        let model = [
          { key: "id", type: "int", props: ["pk", "ai"] },
          { key: "first_name", type: "string" },
          { key: "last_name", type: "string" },
          { key: "email", type: "string" },
          { key: "gender", type: "string" },
          { key: "ip_address", type: "string" }
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
    nSQLonChange: function(data) {
      this.persons = data;
    },
    deletePerson(person) {
      alert(person.id);
    }
  }
};
</script>


<style scoped>
</style>
