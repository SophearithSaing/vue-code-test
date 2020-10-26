<template>
  <nav>
    <v-app-bar app color="primary">
      <v-toolbar-title class="text-uppercase white--text">
        <span class="display-1">News</span>
      </v-toolbar-title>
      <v-spacer></v-spacer>
      <v-text-field
        label="Search"
        solo
        v-model="searchText"
        background-color="white"
        class="my-auto shrink"
        width="200"
      ></v-text-field>
      <v-btn text color="white">
        <v-icon>mdi-magnify</v-icon>
      </v-btn>
    </v-app-bar>
  </nav>
</template>

<script>
export default {
  data() {
    return {
      searchText: "",
    };
  },
  methods: {
  }
};
</script>
