<template>
  <div class="movie-popup" @click="$emit('close')">
    <div class="movie-popup__box" @click.stop>
      <movie :id="id"></movie>
      <button class="movie-popup__close" @click="$emit('close')"></button>
    </div>
    
    <i class="loader"></i>
  </div>
</template>

<script>
import Movie from './Movie.vue';

export default {
  props: ['id'],
  components: { Movie },
  created(){
    window.addEventListener('keyup', function(e){
      if (e.keyCode == 27) {
        this.$emit('close');
      }
    }.bind(this));
  }
}
</script>

<style lang="scss">
@import "./src/scss/variables";
@import "./src/scss/media-queries";
@import "./src/scss/MoviePopup";


</style>
