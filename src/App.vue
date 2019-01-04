<template>
  <v-app>
    <v-toolbar app>
      <v-toolbar-title class="headline text-uppercase">
        <span>Johnny</span>
        <span class="font-weight-light">CSV Loading Example</span>
      </v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn
        flat
        href="https://www.learningsomethingnew.com/vue-js-vue-cli-3-vuetify-cordova-nano-sql-building-a-cross-platform-app-with-a-local-sql-database-that-can-load-data-from-a-static-csv-file"
        target="_blank"
      >
        <span class="mr-2">Tutorial</span>
      </v-btn>
    </v-toolbar>

    <v-content>
      <PersonList/>
    </v-content>
  </v-app>
</template>

<script>
import PersonList from './components/PersonList'

export default {
  name: 'App',
  components: {
    PersonList
  },
  data () {
    return {
      //
    }
  }
}
</script>
