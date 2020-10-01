<template>
  <v-container>
    <ComicComponent />
  </v-container>
</template>

<script>
import axios from "axios";

export default {
  name: "Comic",
  components: {
    ComicComponent: () => import("@/components/ComicComponent")
  },
  data() {
    return {
      comic: {
        num: 0
      },
    };
  },
};
</script>
