<template>
  <div id="app">
    <header class="header">
      <h1>
        <img
          class="center"
          :src="require('./assets/Pokédex_logo.webp')"
          style="width: 400px"
        />
      </h1>
      <v-spacer></v-spacer>
      <v-col class="center">
        <v-text-field v-model="searchText">
          <template slot="append">
            <v-icon> mdi-magnify</v-icon>
          </template>
        </v-text-field>
      </v-col>
    </header>

    <v-main>
      <div>
        <v-row>
          <div
            style="margin-left: 5em; margin-top: 3em"
            v-for="(pokemon, index) in resultQuery"
            :key="pokemon.url"
          >
            <DisplayPokemon
              :id="index + 1"
              :name="pokemon.name"
              :url="pokemon.url"
            />
          </div>
          <div id="scroll-trigger" ref="infinitescrolltrigger">
            <i class="fas fa-spinner fa-spin"></i>
          </div>
        </v-row>
      </div>
    </v-main>
  </div>
</template>

<script>
import DisplayPokemon from "./components/DisplayPokemon";
//import axios from "axios";

export default {
  name: "App",

  components: {
    DisplayPokemon,
  },
  data: () => {
    return {
      pokemons: [],
      NextApiLink: "",
      currentUrl: "",
      searchText: null,
    };
  },

  mounted() {
    this.scrollTrigger();
  },

  created() {
    this.currentUrl = "https://pokeapi.co/api/v2/pokemon";
    this.getPokemonsData();
  },
  computed: {
    resultQuery() {
      if (this.searchText) {
        return this.pokemons.filter((pokemon) => {
          return this.searchText
            .toLowerCase()
            .split(" ")
            .every((v) => pokemon.name.toLowerCase().includes(v));
        });
      } else {
        return this.pokemons;
      }
    },
  },
  methods: {
    // retrieve pokemon data according to the current api url
    getPokemonsData() {
      let req = new Request(this.currentUrl);
      fetch(req)
        .then((response) => {
          if (response.status === 200) return response.json();
        })
        .then((data) => {
          this.NextApiLink = data.next;
          data.results.forEach((pokemon) => {
            pokemon.id = pokemon.url
              .split("/")
              .filter(function (part) {
                return !!part;
              })
              .pop();
            this.pokemons.push(pokemon);
          });
        })
        .catch((error) => {
          console.log(error);
          console.log("error");
        });
    },

    //allows dynamic scrolling by charging progressively the pokemons data
    scrollTrigger() {
      const observer = new IntersectionObserver((entries) => {
        entries.forEach((entry) => {
          if (entry.intersectionRatio > 0 && this.NextApiLink) {
            this.goToNextUrl();
          }
        });
      });

      observer.observe(this.$refs.infinitescrolltrigger);
    },

    // change the current API URL
    goToNextUrl() {
      this.currentUrl = this.NextApiLink;
      this.getPokemonsData();
    },
  },
};
</script>

<style lang="scss" scoped>
#app {
  background: radial-gradient(#ee7752, #e73c7e, #23a6d5, #23d5ab);
}

.center {
  display: block;
  margin-left: auto;
  margin-right: auto;
  width: 50%;
}
</style>
