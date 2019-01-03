<template>
  <!-- <v-container>
    <v-layout text-xs-center wrap>
      <v-flex xs12>
        <v-list two-line subheader v-if="persondb.length > 0">
          <v-list-header>{{ persondb.length }}</v-list-header>
          <v-list-tile v-for="(person, index) in persondb" :key="'person'+index">
            <v-list-tile-content>
              <v-list-tile-title v-html="person.first_name"></v-list-tile-title>
              
            </v-list-tile-content>
          </v-list-tile>
        </v-list>
      </v-flex>
    </v-layout>
  </v-container>-->
  <v-layout row>
    <v-flex xs12 sm6 offset-sm3>
      <v-card>
        <v-toolbar color="indigo" dark>
          <v-toolbar-side-icon></v-toolbar-side-icon>
          <v-toolbar-title>Person List</v-toolbar-title>
          <v-spacer></v-spacer>
          <v-btn icon>
            <v-icon>search</v-icon>
          </v-btn>

          <v-btn icon>
            <v-icon>more_vert</v-icon>
          </v-btn>
        </v-toolbar>
        <v-list>
          <v-list-tile v-for="(person, index) in persondb" :key="'person'+index">
            <v-list-tile-avatar>
              <img
                :src="'https://picsum.photos/200/300/?random&r=' + Math.round(Math.random() * 1000)"
              >
            </v-list-tile-avatar>

            <v-list-tile-content>
              <v-list-tile-title v-text="`${person.first_name} ${person.last_name}`"></v-list-tile-title>
              <v-list-tile-sub-title v-html="person.email"></v-list-tile-sub-title>
            </v-list-tile-content>

            <v-list-tile-action>
              <v-icon color="pink">delete</v-icon>
            </v-list-tile-action>
          </v-list-tile>
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
      persondb: []
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
                this.persondb = rows;
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
      this.persondb = data;
    }
  }
};
</script>


<style scoped>
</style>
