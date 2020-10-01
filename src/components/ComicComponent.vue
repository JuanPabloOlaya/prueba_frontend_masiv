<template>
  <v-row align="center">
    <v-col cols="2"></v-col>
    <v-col cols="12" md="8">
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
            <v-col cols="6" md="4" class="order-md-1">
              <v-btn block @click="getPreviousComic">
                <v-icon>
                  mdi-arrow-left
                </v-icon>Previous
              </v-btn>
            </v-col>
            <v-col cols="12" md="4" class="order-last order-md-2">
              <v-btn block @click="getRandomComic">
                Random
              </v-btn>
            </v-col>
            <v-col cols="6" md="4" class="order-md-3">
              <v-btn block @click="getNextComic">
                Next
                <v-icon>
                  mdi-arrow-right
                </v-icon>
              </v-btn>
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
    /**
     * Obtiene un comic por el numero
     * @param Number number -> Numero del comic a buscar
     */
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
    /**
     * Obtiene el numero del ultimo comic
     */
    async getLastComic() {
      await axios
        .get("https://cors-anywhere.herokuapp.com/https://xkcd.com/info.0.json")
        .then((res) => {
          this.last = res.data.num;
        });
    },
    /**
     * Obtiene el comic siguiente al actual
     */
    getNextComic() {
      let next = this.comic.num + 1;
      this.getComic(next);
    },
    /**
     * Obtiene el comic anterior al actual
     */
    getPreviousComic() {
      let previous = this.comic.num - 1;
      this.getComic(previous);
    },
    /**
     * Obtiene un comic al azar
     */
    getRandomComic() {
      let random = this.getRandomIntFromInterval(1, this.last);

      this.getComic(random);
    },
    /**
     * Devuelve un numero aleatorio en un rango de 1 hasta el numero del ultimo comic
     * @param Number min -> Numero minimo del intervalo
     * @param Number max -> Numero maximo del intervalo
     */
    getRandomIntFromInterval(min, max) {
      return Math.floor(Math.random() * (max - min + 1) + min);
    },
  },
  beforeMount() {
    this.getLastComic().then(() => {
      this.getRandomComic();
    });
  },
};
</script>
