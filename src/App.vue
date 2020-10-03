<template>
  <v-app>
    <v-main
      :style="{ background: this.$vuetify.theme.themes['light'].primary }"
    >
      <router-view></router-view>
    </v-main>
  </v-app>
</template>

<script>
export default {
  name: "App",
  data: () => ({
    //
  }),
};
</script>
