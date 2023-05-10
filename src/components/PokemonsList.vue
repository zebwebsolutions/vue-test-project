<template>
  <div>
    <div class="search-box">
      <input type="text" v-model="searchTerm" placeholder="Search for a Pokemon" />
        <div class="filter-icons">
        <i class="filter-icon" @click="toggleDropdown">Filter</i>
        <i class="clear-icon" @click="resetFilters">clear</i>
        <div class="filter-dropdown" v-show="showDropdown">
          <div class="filter-option" v-for="category in categories" :key="category" @click="filterByCategory(category)">
            <span>{{ category }}</span>
          </div>
      </div>
      </div>
    </div>
    <div v-if="!isLoading">
      <ul class="pokemon-list">
        <li v-for="(pokemon, index) in filteredPokemons" :key="index" @click="showDetails(pokemon.id)">
          <div class="lhs">
            <img :src="pokemon.image" :alt="pokemon.name" />
          </div>
          <div class="rhs">
            <h2>{{ pokemon.name }}</h2>
            <div>{{ pokemon.category }}</div>
          </div>
        </li>
      </ul>
    </div>
    <div v-else>
      <div class="preloader">Loading...</div>
    </div>
  </div>
</template>
<script>
import axios from 'axios';
import PokemonDetails from './PokemonDetails.vue';
import filterIcon from '../components/icons/filter_icon.png';
import {debounce} from 'lodash';

export default {
  name: 'PokemonList',
  components: {
    PokemonDetails,
  },
  data() {
    return {
      pokemons: [],
      searchTerm: '',
      selectedCategory: '',
      categories: [],
      showDropdown: false,
      filterIcon: filterIcon,
      isLoading: true,
    };
  },
  mounted() {
    const requests = [];
    for (let pokemonId = 1; pokemonId <= 30; pokemonId++) {
      const request = axios.get(`https://pokeapi.co/api/v2/pokemon/${pokemonId}`)
        .then(response => {
          const pokemon = {
            id: pokemonId,
            name: response.data.name,
            category: response.data.types[0].type.name,
            image: response.data.sprites.front_default,
          };
          this.pokemons.push(pokemon);
          if (!this.categories.includes(pokemon.category)) {
            this.categories.push(pokemon.category);
          }
        })
        .catch(error => {
          console.log(error);
        });
      requests.push(request);
    }
    Promise.all(requests).then(() => {
      setTimeout(() => {
        this.isLoading = false;
      },1000)
    });
  },
  methods: {
    showDetails(pokemonId) {
      this.$router.push({ name: 'PokemonDetails', params: { id: pokemonId } });
    },
    toggleDropdown() {
      this.showDropdown = !this.showDropdown;
    },
    filterByCategory(category) {
      this.selectedCategory = category;
      this.showDropdown = false;
    },
    resetFilters() {
      this.selectedCategory = '';
    },
  },
  computed: {
    filteredPokemons() {
      if (this.isLoading) {
        // Display preloader if data is still loading
        return [];
      }
      const filtered = this.pokemons.filter(pokemon =>
        pokemon.name.toLowerCase().includes(this.searchTerm.toLowerCase())
      );
      if (this.selectedCategory) {
        return filtered.filter(pokemon => pokemon.category === this.selectedCategory);
      } else {
        return filtered;
      }
    },
  },
  watch: {
    searchTerm: debounce(function () {
      this.isLoading = true;
      setTimeout(() => {
        this.isLoading = false;
      }, 1000);
    }, 500),
  },
};
</script>

<style>
ul.pokemon-list {
    list-style-type: none;
    display: grid;
    grid-template-columns: 33% 33% 33%;
    gap: 2em;
}
ul.pokemon-list li {
    display: flex;
    align-items: center;
    background: #eee;
    color: #000;
    border-radius: 6px;
    cursor: pointer;
}
.pokemon-list h2 {
    line-height: 1;
    margin-bottom: 7px;
}
.pokemon-list li .rhs {
    padding: 1em;
}
.pokemon-list li .lhs {
    background-color: #8d8d8d;
    border-radius: 6px 0 0 6px;
}
.search-box {
  margin-bottom: 20px;
}
.search-box input {
  padding: 7px;
  width: 100%;
  max-width: 275px;
}
.search-box {
    display: flex;
}
.search-box input {
    border: 0;
    border-bottom: 1px solid #d1d1d1;
}
.filter-icon,
.clear-icon{
    padding: 7px 15px;
    display: inline-block;
    font-style: normal;
    cursor: pointer;
    border-bottom: 1px solid #d1d1d1;
}
:focus-visible {
    outline: unset;
}
.filter-icon {
    position: relative;
    background: url(/filter_icon.png) right center no-repeat;
    background-size: 18px;
    padding-right: 20px;
}
.search-box input {
    background: url(search_icon.png) no-repeat left center;
    background-size: 18px;
    padding-left: 22px;
}
.search-box input:focus {
    background: unset;
    padding-left: 0;
}
.filter-dropdown {
    position: absolute;
    top: 38px;
    left: 0;
    z-index: 9999;
    background: #d1d1d1;
    width: 100%;
    max-width: 120px;
}
.filter-dropdown span {
    border-bottom: 1px solid #000;
    display: block;
    padding: 5px 10px;
    cursor: pointer;
}
.filter-dropdown .filter-option:last-of-type span {
  border-bottom: unset;
}
.filter-dropdown span:hover {
  background-color: #eee;
}
.preloader {
  position: fixed;
  left: 0;
  top: 0;
  z-index: 9999;
  width: 100%;
  height: 100%;
  overflow: visible;
  text-align: center;
  background: #fff url('loader.gif') no-repeat center center;
}

/* Styles for the spinner */
.preloader .spinner {
  position: absolute;
  left: 50%;
  top: 50%;
  width: 60px;
  height: 60px;
  margin: -30px 0 0 -30px;
  border: 6px solid #ccc;
  border-radius: 50%;
  border-top-color: #3498db;
  animation: spin 1s ease-in-out infinite;
}

/* Keyframe animation for the spinner */
@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
</style>
