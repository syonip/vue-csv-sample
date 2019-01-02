<template>
  <div>
    {{ response }}
  </div>
</template>

<script>
const axios = require('axios');

export default {
  name: 'PersonList',
  data() {
    return {
      response: ''
    }
  },
  mounted() {
    document.addEventListener(
      typeof cordova !== 'undefined' ? 'deviceready' : 'DOMContentLoaded',
      () => {
        axios
          .get('MOCK_DATA.csv')
          .then(response => {
            this.response = response;
          })
          .catch(err => {
            this.response = `ERROR! ${err}`;
          });
      }
    );
  }
};
</script>


<style scoped>

</style>
