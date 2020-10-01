<template>
  <v-row align="center">
    <v-col cols="3"></v-col>
    <v-col cols="12" sm="6">
      <v-card>
        <v-card-title>
          {{ comic.title }}
        </v-card-title>
        <v-card-subtitle>
          {{ comic.alt }}
        </v-card-subtitle>
        <v-card-text>
          <v-row>
            <v-col>
              <v-card>
                <v-img :src="comic.img"></v-img>
              </v-card>
              <p class="font-italic text-right">
                "{{ comic.safe_title }}" - {{ comic.year }}
              </p>
            </v-col>
          </v-row>
          <v-row>
            <v-col>
              {{ comic }}
            </v-col>
          </v-row>
        </v-card-text>
      </v-card>
    </v-col>
  </v-row>
</template>

<script>
import axios from "axios";

export default {
  name: "ComicComponent",
  data() {
    return {
      comic: {
        alt: "",
        day: "",
        img: "",
        link: "",
        month: "",
        news: "",
        num: "",
        safe_title: "",
        title: "",
        transcript: "",
        year: "",
      },
      last: "",
    };
  },
  methods: {
    async getComic(number) {
      await axios
        .get(
          "https://cors-anywhere.herokuapp.com/https://xkcd.com/" +
            number +
            "/info.0.json"
        )
        .then((res) => {
          this.comic = res.data;
        })
        .catch(console.log);
    },
    async getLastComic() {
      await axios
        .get("https://cors-anywhere.herokuapp.com/https://xkcd.com/info.0.json")
        .then((res) => {
          this.last = res.data.num;
        });
    },
    getRandomIntFromInterval(min, max) {
      return Math.floor(Math.random() * (max - min + 1) + min);
    },
  },
  beforeMount() {
    this.getLastComic().then(() => {
      let comicId = this.getRandomIntFromInterval(1, this.last);

      this.getComic(comicId)
    });
  },
};
</script>
