<template>
  <v-app>
    <v-app-bar color="blue" app dark>
      <v-toolbar-title>My Custom Knack App</v-toolbar-title>
    </v-app-bar>
    <v-content>
      <v-container>
        <nuxt />
      </v-container>
    </v-content>
    <v-footer
      :fixed="fixed"
      app
    >
      <span>&copy; 2019</span>
    </v-footer>
  </v-app>
</template>

<script>
export default {
  data () {
    return {
      fixed: false,
      title: 'Vuetify.js'
    }
  }
}
</script>
