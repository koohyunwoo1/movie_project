<template>
  <h1 style=" margin-top: 50px; margin-bottom: 0px; text-align: center;">원하는 태그를 선택하고 영화를 추천받으세요 !</h1>
  <div class="container">
    <h2 style="margin-bottom: 30px; margin-left: 22px; font-size: 30px;">장르</h2>
    <div>
      <div v-if="genres">
        <span v-for="(genre, index) in genres" :key="index">
          <label :class="{ 'selected': selectedGenres.includes(genre.pk)}" @click="selectGenre(genre.pk)">
            {{ genre.name }}
          </label>
        </span>
      </div>
      <p v-else>No genres available</p>
    </div>
  </div>

  <div class="container">
    <div style="margin-left: 0px; margin-bottom: 22px;">

      <h2 style="display: inline; margin-right: 22px; margin-bottom: 30px; margin-left: 22px; font-size: 30px;">무드</h2> 
      <p style="display: inline; margin-bottom: 100px;">(최대 2개 선택)</p>
    </div>
    <div>
      <div v-if="moods">
        <span v-for="(mood, index) in moods" :key="index">
          <label :class="{ 'selected': selectedMoods.includes(mood.pk)}" @click="toggleMood(mood.pk)" >
            {{ mood.name }}
          </label>
        </span>
      </div>
      <p v-else>No moods available</p>
    </div>
  </div>

  <div style="margin-right: 50px;">
    <div class="button-container">
      <button class="button" @click="fetchMovies" :disabled="selectedGenres.length === 0 && selectedMoods.length === 0">Go</button>
    </div>
  </div>
  

  <div>
    <div v-if="movies.length">
      <h2 style="margin-left: 50px;">선택된 장르 또는 무드의 영화</h2>
      <div class="movie-img">
        <div v-for="(movie, index) in movies.slice(0, 5)" :key="index" class="movie-item">
          <RouterLink :to="{ name: 'MovieDetailView', params: { id: movie.pk }}">
            <img :src="'http://image.tmdb.org/t/p/w500' + movie.poster_path" :alt="movie.title" class="movie-image"/>
          </RouterLink>
        </div>
      </div>
    </div>


    <div v-if="showNoMoviesMessage">
      <div style="margin-left: 50px;">
        <h3>선택하신 장르나 무드에 해당하는 영화를 찾지 못하였습니다.</h3>
      </div>
    </div>    
  </div>
  
  <RouterView/> 
</template>

<script setup>
import axios from 'axios'
import { ref } from 'vue'
import { useMovieStore } from '@/stores/modules/genres_and_moods'
import { RouterLink, RouterView } from 'vue-router'


const store = useMovieStore()
const movies = ref([])
const genres = ref([])
const selectedGenres = ref([])
const moods = ref([])
const selectedMoods = ref([])
const showNoMoviesMessage = ref(false)


const genreslist = () => {
  axios.get(`${store.API_URL}/api/v1/movies/genre/`, {
    headers: { Authorization: `Token ${store.token}` }
  })
  .then(res => {
    genres.value = res.data;
  })
  .catch(err => {
    console.error(err)
  });
}

const moodslist = () => {
  axios.get(`${store.API_URL}/api/v1/movies/mood/`, {
    headers: { Authorization: `Token ${store.token}` }
  })
  .then(res => {
    moods.value = res.data;
  })
  .catch(err => {
    console.error(err)
  });
}

const selectGenre = (genrePk) => {
  if (selectedGenres.value.includes(genrePk)) {
    selectedGenres.value = selectedGenres.value.filter(pk => pk !== genrePk);
  } else {
    selectedGenres.value.push(genrePk);
  }
}

const toggleMood = (moodPk) => {
  if (selectedMoods.value.includes(moodPk)) {
    selectedMoods.value = selectedMoods.value.filter(pk => pk !== moodPk);
  } else if (selectedMoods.value.length < 2) {
    selectedMoods.value.push(moodPk);
  }
}

const fetchMovies = () => {
  if (selectedGenres.value.length === 0 && selectedMoods.value.length === 0) return;

  const queryString = new URLSearchParams();
  selectedGenres.value.forEach(genrePk => {
    queryString.append('genre_pk', genrePk);
  });
  selectedMoods.value.forEach(moodPk => {
    queryString.append('mood_pk', moodPk);
  });

  axios.get(`${store.API_URL}/api/v1/movies/filter/?${queryString.toString()}`, {
    headers: { Authorization: `Token ${store.token}` }
  })
  .then(response => {
    movies.value = response.data;
    showNoMoviesMessage.value = movies.value.length === 0;
    selectedGenres.value = [];
    selectedMoods.value = [];
    scrollExactMatchesIntoView()
  })
  .catch(err => {
    console.error(err);
  });
}

genreslist()
moodslist()

const scrollExactMatchesIntoView = () => {
  const centerElement = document.querySelector('.movie-img');
  if (centerElement) {
    centerElement.scrollIntoView({ behavior: 'smooth'});
  } else {
    setTimeout(scrollExactMatchesIntoView, 10)
  }
}

</script>

<style scoped>

h1 {
  margin-left: 22px;
}



label {
  padding: 8px 16px;
  margin: 5px;
  border: none;
  background-color: transparent;
  cursor: pointer;
  max-width: fit-content;
  border-radius: 5px;
  border: solid 1px #87CEEB;
  /* border: solid 1px gray; */
}

.button {
  margin-top: 50px;
}

.selected {
  background-color: #BBD0E9;
}

.container {
  margin-left: 50px;
  margin-top: 110px;
}

.button-container {
  text-align: right;
}


.movie-img {
  margin-top: 50px;
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  justify-content: center;
}

.movie-item {
  margin: 10px;
  transform: scale(0.9);
  transition: transform 0.3s ease-in-out; 
}

.movie-item:hover {
  transform: scale(1); 
  filter: brightness(1.5); 
}

.movie-image {
  width: 300px;
  border-radius: 15px;
}
</style>