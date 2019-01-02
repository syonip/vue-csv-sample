<template>
  <div>
    {{ persondb.length }}
  </div>
</template>

<script>
const axios = require('axios');
import { nSQL } from "nano-sql";

export default {
  name: 'PersonList',
  data() {
    return {
      persondb: []
    }
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
