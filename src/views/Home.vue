<template>
  <div class="home">
    <v-snackbar v-model="snackbar" :timeout="3000" color="primary" top>
      <span>My Vue.js knowledge is currently limited.</span>
      <template v-slot:action="{ attrs }">
        <v-btn color="white" text v-bind="attrs" @click="snackbar = false">
          Close
        </v-btn>
      </template>
    </v-snackbar>
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
      <v-btn text color="white" @click="search">
        <v-icon>mdi-magnify</v-icon>
      </v-btn>
    </v-app-bar>

    <div class="text-center" v-if="loading">
      <v-progress-circular
        :size="100"
        :width="10"
        indeterminate
        color="primary"
      ></v-progress-circular>
    </div>

    <div v-else>
      <v-row>
        <v-col>
          <h1 class="display-1 blue--text mx-5">Headline</h1>
        </v-col>
        <v-col class="text-right mx-5">
          <v-menu offset-y>
            <template v-slot:activator="{ on, attrs }">
              <v-btn class="mx-5" color="error" @click="makeWrongAPICall">
                Make Wrong API Call
              </v-btn>
              <v-btn color="primary" dark v-bind="attrs" v-on="on">
                Filter
              </v-btn>
            </template>
            <v-list>
              <v-list-item v-for="(source, index) in sources" :key="index">
                <v-btn depressed @click="selectSource(source.name)">
                  {{ source.name }}
                </v-btn>
              </v-list-item>
            </v-list>
          </v-menu>
        </v-col>
      </v-row>

      <v-container>
        <v-row>
          <v-col
            cols="12"
            sm="6"
            md="4"
            lg="3"
            v-for="(article, index) in filteredArticles"
            :key="index"
          >
            <v-dialog
              v-model="dialog"
              width="500"
              transition="dialog-bottom-transition"
            >
              <template v-slot:activator="{ on, attrs }">
                <v-card max-height="400" color="grey lighten-1" dark href="#!">
                  <v-img
                    :src="article.urlToImage"
                    height="250"
                    gradient="rgba(0, 0, 0, .50), rgba(0, 0, 0, 1)"
                  >
                    <v-row class="fill-height text-right ma-0">
                      <v-col cols="12">
                        <v-chip
                          label
                          class="mx-0 mb-2 text-uppercase"
                          color="grey darken-3"
                          text-color="white"
                          small
                        >
                          {{ article.publishedAt }}
                        </v-chip>

                        <h3 class="title font-weight-bold mb-2">
                          {{
                            article.title && article.title.length > 60
                              ? article.title.substring(0, 60) + "..."
                              : article.title
                          }}
                        </h3>

                        <p class="caption">
                          {{
                            article.author && article.author.length > 40
                              ? article.author.substring(0, 40) + "..."
                              : article.author
                          }}
                        </p>
                      </v-col>

                      <v-col align-self="end">
                        <v-chip
                          class="text-uppercase ma-0"
                          color="primary"
                          label
                          small
                          v-bind="attrs"
                          v-on="on"
                          @click="selectArticle(index)"
                        >
                          Read More
                        </v-chip>
                      </v-col>
                    </v-row>
                  </v-img>
                </v-card>
              </template>
              <v-card>
                <v-toolbar dark color="primary">
                  <v-btn icon dark @click="dialog = false">
                    <v-icon>mdi-close</v-icon>
                  </v-btn>
                </v-toolbar>
                <v-img :src="selectedArticle.urlToImage"></v-img>
                <v-card-title>
                  {{ selectedArticle.title }}
                </v-card-title>
                <v-card-text>
                  {{ selectedArticle.description }}
                </v-card-text>
                <v-divider></v-divider>

                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn
                    color="primary"
                    text
                    @click="dialog = false"
                    :href="selectedArticle.url"
                    target="_blank"
                  >
                    Read full article
                  </v-btn>
                </v-card-actions>
              </v-card>
            </v-dialog>
          </v-col>
        </v-row>
      </v-container>
    </div>

    <div v-if="filteredArticles.length == 0 && error === false">
      <h1 class="display-3 text-center">No news from selected source.</h1>
    </div>

    <div v-if="error">
      <h1 class="display-3 text-center">Failed to fetch data.</h1>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import moment from "moment";
// import Navbar from "@/components/Navbar";

export default {
  // components: {
  //   Navbar,
  // },
  data() {
    return {
      dialog: false,
      loading: true,
      error: false,
      snackbar: true,
      apiKey: "099148be22804e849a0c6fe022b7cf5e",
      articles: [],
      sources: [],
      source: "",
      searchText: "",
      selectedArticle: {},
    };
  },
  mounted() {
    axios
      .get(
        `https://newsapi.org/v2/top-headlines?country=us&apiKey=${this.apiKey}`
      )
      .then((response) => {
        this.articles = response.data.articles;
        console.log(response.data);
        this.articles.forEach((article) => {
          article.publishedAt = moment(article.publishedAt).format(
            "Do MMM YYYY"
          );
        });
      })
      .catch((error) => {
        console.log(error);
        this.error = true;
      })
      .finally(() => (this.loading = false));

    axios
      .get(`https://newsapi.org/v2/sources?apiKey=${this.apiKey}`)
      .then((response) => {
        this.sources = response.data.sources;
        console.log(response.data);
      })
      .catch((error) => {
        console.log(error);
        this.error = true;
      });
  },
  methods: {
    search() {
      axios
        .get(
          `https://newsapi.org/v2/top-headlines?q=${this.searchText}&apiKey=${this.apiKey}`
        )
        .then((response) => {
          this.articles = response.data.articles;
          console.log(this.searchText);
          console.log(response.data);
          this.articles.forEach((article) => {
            article.publishedAt = moment(article.publishedAt).format(
              "Do MMM YYYY"
            );
          });
        });
    },
    selectArticle(index) {
      this.selectedArticle = this.articles[index];
    },
    selectSource(source) {
      this.source = source;
    },
    makeWrongAPICall() {
      this.articles = [];
      axios
        .get("https://newsapi.org/v2/sources?apiKey")
        .then((response) => {
          this.sources = response.data.sources;
          console.log(response.data);
        })
        .catch((error) => {
          console.log(error);
          this.error = true;
        });
    },
  },
  computed: {
    filteredArticles() {
      if (this.source !== "") {
        return this.articles.filter((article) => {
          return article.source.name === this.source;
        });
      } else {
        return this.articles;
      }
    },
  },
};
</script>

<style scoped>
.v-progress-circular {
  margin-top: 20%;
}
</style>
