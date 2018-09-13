<template>
  <section class="movie">
    <div class="movie__container" v-if="movieLoaded">
      <header class="movie__header" :class="{'movie__header--page': type=='page'}" :style="{ 'background-image': 'url(' + movieBackdropSrc + ')' }">
        <div class="movie__wrap movie__wrap--header" :class="{'movie__wrap--page': type=='page'}">
          <figure class="movie__poster">
            <img v-if="moviePosterSrc" class="movie__img" src="~assets/placeholder.png" v-img="moviePosterSrc">
            <img v-if="!moviePosterSrc" class="movies-item__img is-loaded" src="~assets/no-image.png">
          </figure>
          <div class="movie__title">
            <h1 class="movie__title-text">
              {{ movie.title }}
              <span v-if="movie.tagline">{{ movie.tagline }}</span>
            </h1>
          </div>
        </div>
      </header>
      <div class="movie__main">
        <div class="movie__wrap movie__wrap--main" :class="{'movie__wrap--page': type=='page'}">
          <div class="movie__actions" v-if="userLoggedIn && favoriteChecked">
            <a href="#" class="movie__actions-link" :class="{'active' : favorite === true}" @click.prevent="toggleFavorite">
              <svg class="movie__actions-icon" :class="{'waiting' : favorite === ''}">
                <use xlink:href="#iconFavorite"></use>
              </svg>
              <span class="movie__actions-text" v-if="favorite === ''">Wait...</span>
              <span class="movie__actions-text" v-else-if="favorite">Marked as Favorite</span>
              <span class="movie__actions-text" v-else>Mark as Favorite?</span>
            </a>
          </div>
          <div class="movie__info">
            <div v-if="movie.overview" class="movie__description">
              {{ movie.overview }}
            </div>
            <div class="movie__details">
              <div v-if="movie.genres.length" class="movie__details-block">
                <h2 class="movie__details-title">
                  Genres
                </h2>
                <div class="movie__details-text">
                  {{ nestedDataToString(movie.genres) }}
                </div>
              </div>
              <div v-if="movie.release_date" class="movie__details-block">
                <h2 class="movie__details-title">
                  Release Date
                </h2>
                <div class="movie__details-text" v-formatDate="movie.release_date"></div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
import axios from 'axios'
import storage from '../storage.js'
import img from '../directives/v-image.js'
import formatDate from '../directives/v-formatDate.js'

export default {
  props: ['id', 'type'],
  directives: {
    img: img,
    formatDate: formatDate
  },
  data(){
    return{
      movie: {},
      movieLoaded: false,
      moviePosterSrc: '',
      movieBackdropSrc: '',
      userLoggedIn: storage.sessionId ? true : false,
      favoriteChecked: false,
      favorite: ''
    }
  },
  // computed: {
  //   loaded(){
  //     return this.movieLoaded ? true : false;
  //   }
  // },
  methods: {
    fetchMovie(id){
      axios.get(`https://api.themoviedb.org/3/movie/${id}?api_key=${storage.apiKey}&language=en-US`)
      .then(function(resp){
          let movie = resp.data;
          this.movie = movie;
          this.poster();
          this.backdrop();
          if(this.userLoggedIn){
            this.checkIfInFavorites(movie.id);
          } else {
            this.movieLoaded = true;
          }
          // Push state
          if(storage.createMoviePopup){
            storage.moviePath = '/movie/' + id;
            history.pushState({ popup: true }, null, storage.moviePath);
            storage.createMoviePopup = false;
          }
          // Change Page title
          document.title = this.movie.title + storage.pageTitlePostfix;
      }.bind(this))
      .catch(function(error) {
        this.$router.push({ name: '404' });
      }.bind(this));
    },
    poster() {
      if(this.movie.poster_path){
        this.moviePosterSrc = 'https://image.tmdb.org/t/p/w600_and_h900_bestv2' + this.movie.poster_path;
      }
    },
    backdrop(){
      if(this.movie.backdrop_path){
        this.movieBackdropSrc = 'https://image.tmdb.org/t/p/w500' + this.movie.backdrop_path;
      }
    },
    nestedDataToString(data) {
      let nestedArray = [], resultString;
      data.forEach((item) => nestedArray.push(item.name));
      resultString = nestedArray.join(', ');
      return resultString;
    },
    checkIfInFavorites(id){
      axios.get(`https://api.themoviedb.org/3/movie/${id}/account_states?api_key=${storage.apiKey}&session_id=${storage.sessionId}`)
      .then(function(resp){
          this.favorite = resp.data.favorite;
          this.favoriteChecked = true;
          this.movieLoaded = true;
      }.bind(this))
    },
    toggleFavorite(){
      let favoriteInvert = !this.favorite;
      this.favorite = '';
      axios.post(`https://api.themoviedb.org/3/account/${storage.userId}/favorite?api_key=${storage.apiKey}&session_id=${storage.sessionId}`, {
        'media_type': 'movie',
        'media_id': this.id,
        'favorite': favoriteInvert
      })
      .then(function(resp){
        this.favorite = favoriteInvert;
        eventHub.$emit('updateFavorite');
      }.bind(this));
    }
  },
  watch: {
    id: function(val){
      this.fetchMovie(val);
    }
  },
  created(){
    this.fetchMovie(this.id);
  }
}
</script>

<style lang="scss">
@import "./src/scss/variables";
@import "./src/scss/media-queries";
@import "./src/scss/Movie";


</style>
