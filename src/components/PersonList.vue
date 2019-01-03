<template>
  <v-layout row>
    <v-snackbar v-model="showOperationStatus" left>{{operationStatus}}</v-snackbar>
    <v-flex xs12 sm6 offset-sm3>
      <v-card>
        <v-toolbar color="cyan" dark>
          <v-toolbar-side-icon></v-toolbar-side-icon>

          <v-toolbar-title>Person List</v-toolbar-title>

          <v-spacer></v-spacer>

          <v-btn icon ripple @click="deleteAll()">
            <v-icon color="lighten-1">delete</v-icon>
          </v-btn>
        </v-toolbar>

        <v-list three-line>
          <v-subheader>
            {{persons.length}} records
            <v-spacer></v-spacer>
            <v-btn @click="loadCsv">load csv</v-btn>
          </v-subheader>
          <template v-for="(person, index) in persons">
            <v-divider :inset="true" :key="index"></v-divider>

            <v-list-tile :key="'person'+index" avatar>
              <v-list-tile-avatar>
                <img :src="person.avatar">
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
const axios = require('axios');
import { nSQL } from 'nano-sql';
// import { getMode } from 'cordova-plugin-nano-sqlite/lib/sqlite-adapter'; //fix path

export default {
  name: 'PersonList',
  data() {
    return {
      showOperationStatus: false,
      operationStatus: '',
      persons: []
    };
  },
  mounted() {
    document.addEventListener(
      typeof cordova !== 'undefined' ? 'deviceready' : 'DOMContentLoaded',
      () => {
        let model = [
          { key: 'id', type: 'int', props: ['pk', 'ai'] },
          { key: 'first_name', type: 'string' },
          { key: 'last_name', type: 'string' },
          { key: 'email', type: 'string' },
          { key: 'gender', type: 'string' },
          { key: 'ip_address', type: 'string' },
          { key: 'avatar', type: 'string' }
        ];

        if (window.nSQLite && window.cordova.platformId != 'browser') {
          nSQL('persondb')
            .model(model)
            .config({
              mode: window.nSQLite.getMode() // required
            })
            .connect();
        } else {
          nSQL('persondb')
            .config({
              mode: 'PERM'
            })
            .model(model)
            .connect();
        }

        nSQL().onConnected(() => {
          this.refreshData();
        });
      }
    );
  },
  methods: {
    async refreshData() {
      let rows = await nSQL('persondb')
        .query('select')
        .exec();
      if (rows.length == 0) this.persons = [];
      // else this.persons = rows.filter(x => x.first_name);
      else this.persons = rows;
    },
    loadCsv() {
      axios.get('MOCK_DATA.csv', {}).then(response => {
        nSQL()
          .loadCSV('persondb', response.data, false)
          .then(() => {
            this.refreshData().then(() => {
              this.showOperationStatus = true;
              this.operationStatus = `CSV loaded`;
            });
          });
      });
    },
    deletePerson(person) {
      nSQL('persondb')
        .query('delete')
        .where(['id', '=', person.id])
        .exec()
        .then(rows => {
          this.showOperationStatus = true;
          this.operationStatus = `${rows.length} rows deleted`;
          this.refreshData();
        });
    },
    deleteAll(person) {
      nSQL('persondb')
        .query('delete')
        .exec()
        .then(result => {
          if (result[0].msg) {
            this.showOperationStatus = true;
            this.operationStatus = result[0].msg;
          }
          this.refreshData();
        });
    }
  }
};
</script>


<style scoped>
</style>
