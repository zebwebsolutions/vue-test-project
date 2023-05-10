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
</template>

<script>
import axios from 'axios';
import PokemonDetails from './PokemonDetails.vue';
import filterIcon from '../components/icons/filter_icon.png';

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
      filterIcon: filterIcon
    };
  },
  mounted() {
    for (let pokemonId = 1; pokemonId <= 30; pokemonId++) {
      axios.get(`https://pokeapi.co/api/v2/pokemon/${pokemonId}`)
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
    }
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
    background: url("~filterIcon");
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
</style>
