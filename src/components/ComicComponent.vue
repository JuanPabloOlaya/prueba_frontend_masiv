<template>
  <v-row align="center">
    <v-col cols="2"></v-col>
    <v-col cols="12" md="8">
      <v-expand-transition>
        <v-card v-show="expand">
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
                  <v-img
                    :src="comic.img"
                    :lazy-src="comic.img"
                    height="500"
                    contain
                  ></v-img>
                </v-card>
                <p class="font-italic text-right">
                  "{{ comic.safe_title }}" - {{ comic.year }}
                </p>
              </v-col>
            </v-row>
            <v-row>
              <v-col cols="2"></v-col>
              <v-col cols="8" class="text-center">
                <v-rating large v-model="rating" color="accent"></v-rating>
              </v-col>
            </v-row>
            <v-row>
              <v-col cols="6" md="4" class="order-md-1">
                <v-btn
                  block
                  @click="getPreviousComic"
                  color="secondary"
                  v-if="comic.num > 0"
                >
                  <v-icon> mdi-arrow-left </v-icon>Previous
                </v-btn>
              </v-col>
              <v-col cols="12" md="4" class="order-last order-md-2">
                <v-btn block @click="getRandomComic" color="primary">
                  Random
                </v-btn>
              </v-col>
              <v-col cols="6" md="4" class="order-md-3">
                <v-btn
                  block
                  @click="getNextComic"
                  color="secondary"
                  v-if="comic.num <= last"
                >
                  Next
                  <v-icon>
                    mdi-arrow-right
                  </v-icon>
                </v-btn>
              </v-col>
            </v-row>
          </v-card-text>
        </v-card>
      </v-expand-transition>
      <v-dialog
        v-model="dialog.show"
        :persistent="dialog.loading"
        max-width="600px"
      >
        <v-card>
          <v-card-title>
            <span class="headline" v-if="dialog.title != ''">
              {{ dialog.title }}
            </span>
          </v-card-title>
          <v-card-text v-if="dialog.loading">
            Cargando...
            <v-progress-linear
              indeterminate
              color="primary"
              class="mb-0"
            ></v-progress-linear>
          </v-card-text>
          <v-card-title v-if="dialog.error">
            <h3>Se ha generado un problema</h3>
          </v-card-title>
          <v-card-subtitle v-if="dialog.error">
            {{ dialog.errorMessage }}
          </v-card-subtitle>
        </v-card>
      </v-dialog>
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
        rating: 0,
      },
      last: "",
      rating: 0,
      expand: true,
      dialog: {
        title: "",
        error: false,
        errorMessage: "",
        show: false,
        description: "",
        loading: false,
      },
    };
  },
  methods: {
    /**
     * Obtiene un comic por el numero
     * @param Number number -> Numero del comic a buscar
     */
    async getComic(number) {
      this.dialog.show = true;
      this.dialog.loading = true;
      let comics = JSON.parse(localStorage.getItem("comics"));
      let found = false;

      for (let i = 0; i < comics.length && !found; i++) {
        if (comics[i].num == number) {
          this.comic = comics[i];
          this.rating = comics[i].rating;
          found = true;
          this.expand = true;
          this.resetDialog();
        }
      }

      if (!found) {
        await axios
          .get(
            "https://cors-anywhere.herokuapp.com/https://xkcd.com/" +
              number +
              "/info.0.json"
          )
          .then((res) => {
            this.comic = res.data;
            this.comic.rating = 0;
            this.rating = this.comic.rating;
            comics.push(this.comic);

            localStorage.setItem("comics", JSON.stringify(comics));
            this.expand = true;
            this.resetDialog();
          })
          .catch((error) => {
            console.log(error.response);
            this.resetDialog();
          });
      }
    },
    /**
     * Obtiene el numero del ultimo comic
     */
    async getLastComic() {
      this.dialog.loading = true;
      this.dialog.show = true;
      await axios
        .get("https://cors-anywhere.herokuapp.com/https://xkcd.com/info.0.json")
        .then((res) => {
          this.last = res.data.num;
          this.resetDialog();
        })
        .catch((error) => {
          console.log(error.response);

          if (error.response.status == 429) {
            this.dialog.show = true;
            this.dialog.error = true;
            this.dialog.errorMessage = "Maximun number of request was exceeded";
          }
        });
    },
    /**
     * Obtiene el comic siguiente al actual
     */
    getNextComic() {
      this.expand = false;

      let next = this.comic.num + 1;
      this.getComic(next);
    },
    /**
     * Obtiene el comic anterior al actual
     */
    getPreviousComic() {
      this.expand = false;

      let previous = this.comic.num - 1;
      this.getComic(previous);
    },
    /**
     * Obtiene un comic al azar
     */
    getRandomComic() {
      this.expand = false;

      let random = this.getRandomIntFromInterval(1, this.last);

      this.getComic(random);
    },
    /**
     * Actualiza la calificación del comic en el arreglo
     */
    qualifyComic() {
      let comics = JSON.parse(localStorage.getItem("comics"));
      let updated = false;

      for (let i = 0; i < comics.length && !updated; i++) {
        if (comics[i].num == this.comic.num) {
          comics[i] = this.comic;
          updated = true;
        }
      }
      localStorage.setItem("comics", JSON.stringify(comics));
    },
    /**
     * Devuelve un numero aleatorio en un rango de 1 hasta el numero del ultimo comic
     * @param Number min -> Numero minimo del intervalo
     * @param Number max -> Numero maximo del intervalo
     */
    getRandomIntFromInterval(min, max) {
      return Math.floor(Math.random() * (max - min + 1) + min);
    },
    /**
     * Restablece las propiedades del dialogo a sus propiedades por defecto
     */
    resetDialog() {
      this.dialog.show = false;
      (this.dialog.title = ""), (this.dialog.description = "");
      this.dialog.loading = false;
      this.dialog.error = false;
      this.dialog.errorMessage = "";
    },
  },
  beforeMount() {
    localStorage.setItem("comics", JSON.stringify([]));
    this.expand = false;
    this.getLastComic().then(() => {
      this.getRandomComic();
    });
  },
  watch: {
    rating: function() {
      this.comic.rating = this.rating;
      this.qualifyComic();
    },
  },
};
</script>
