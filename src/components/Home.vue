<template>
  <section class="home">
    <!-- <header class="home__header"> -->
      <!-- <div class="home__header-wrap">
        <h1 class="home__header-title">The Movie DB App</h1>
        <strong class="home__header-subtitle">Made with Vue.js</strong>
        <a href="https://github.com/dmtrbrl/tmdb-app" target="_blank" class="home__header-link">
          <svg class="home__header-link-icon">
            <use xlink:href="#iconGithub"></use>
          </svg>
          <span>View Code</span>
        </a>
      </div> -->
    <!-- </header> -->
    <movies-list v-for="item in listTypes" v-if="item.isCategory" :type="'component'" :mode="'collection'" :category="item.query" :shortList="true"></movies-list>
  </section>
</template>

<script>
import axios from 'axios'
import storage from '../storage.js'
import MoviesList from './MoviesList.vue'

export default {
  components: { MoviesList },
  data(){
    return {
      listTypes: storage.listTypes
    }
  },
  created(){
    document.title = 'FrontEnd';
    storage.backTitle = document.title;
  }
}
</script>

<style lang="scss">
@import "./src/scss/variables";
@import "./src/scss/media-queries";
@import "./src/scss/Home";

</style>
