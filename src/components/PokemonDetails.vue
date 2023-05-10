<template>
  <div>
    <h3 class="page-heading">Details Page</h3>
    <div v-if="pokemonDetails" class="details-wrap">
      <div class="details-lhs">
        <h2 class="pokemon-name">{{ pokemonDetails.name }}</h2>
        <img :src="pokemonDetails.image" :alt="pokemonDetails.name" />
        <p>Weight: {{ pokemonDetails.weight }} Kg</p>
        <p>Height: {{ pokemonDetails.height }} m</p>
        <div class="category-wrap">{{ pokemonDetails.category }}</div>
      </div>
      <div class="details-rhs">
        <PokemonStats :stats="pokemonDetails.stats" />
      </div>
    </div>
    <div class="evolutions-wrap">
      <h3>Evolution Forms</h3>
      <div v-if="evolutionChain" class="evolutions">
        <div class="flex-wrap">
          <div v-for="(evolution, index) in evolutionChain" :key="index" class="evolution">
            <img :src="evolution.image" :alt="evolution.name" />
            <div>{{ evolution.name }}</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import axios from 'axios';
import PokemonStats from './PokemonStats.vue';
import { useRoute } from 'vue-router';

export default {
  name: 'PokemonDetails',
  data() {
    return {
      pokemonDetails: null,
      evolutionChain: null,
    };
  },
  components: {
    PokemonStats,
  },
  mounted() {
    const route = useRoute();
    this.id = route.params.id;
    axios.get(`https://pokeapi.co/api/v2/pokemon/${this.id}`)
      .then(response => {
        const pokemonDetails = {
          name: response.data.name,
          category: response.data.types[0].type.name,
          image: response.data.sprites.other.dream_world.front_default,
          weight: response.data.weight,
          height: response.data.height,
          stats: response.data.stats
        };
        this.pokemonDetails = pokemonDetails;
        this.fetchEvolutionChain(response.data);
      })
      .catch(error => {
        console.log(error);
      });
  },
  methods: {

    fetchEvolutionChain(pokemon) {
      axios.get(pokemon.species.url)
  .then(response => {
    const speciesData = response.data;
    const evolutionChainUrl = speciesData.evolution_chain.url;
    return axios.get(evolutionChainUrl);
  })
  .then(response => {
    const evolutionChainData = response.data.chain;
    let evolutionChain = [];

    // Find the first evolution of the current Pokemon
    let currentEvolution = evolutionChainData;
    while (currentEvolution.species.name !== pokemon.name && currentEvolution.evolves_to.length > 0) {
      currentEvolution = currentEvolution.evolves_to[0];
    }
    if (currentEvolution.species.name !== pokemon.name) {
      // The current Pokemon is not in the evolution chain
      return;
    }

    // Add the rest of the Pokemon in the evolution chain
    while (currentEvolution.evolves_to.length > 0) {
      currentEvolution = currentEvolution.evolves_to[0];
      const pokemonName = currentEvolution.species.name;
      const pokemonImage = `https://img.pokemondb.net/artwork/${pokemonName}.jpg`;
      evolutionChain.push({ name: pokemonName, image: pokemonImage });
    }

    // Save the evolution chain to the data property
    this.evolutionChain = evolutionChain;
  })
  .catch(error => {
    console.log(error);
  });

}




  },
};
</script>

<style>
.page-heading {
    font-size: 42px;
    margin-top: 40px;
    margin-bottom: 40px;
    font-weight: 600;
}
.details-wrap {
    display: grid;
    grid-template-columns: 50% 50%;
}
.details-wrap .details-rhs canvas {
    width: 500px !important;
    height: 500px !important;;
}
.pokemon-name {
    font-size: 36px;
    font-weight: 500;
    margin-bottom: 20px;
}
.category-wrap {
    background: #eec561;
    display: inline-block;
    padding: 4px 8px;
    color: #fff;
    text-transform: capitalize;
    margin-top: 16px;
}
.flex-wrap {
    display: flex;
    justify-content: space-around;
    text-align: center;
}
.flex-wrap img {
    max-width: 125px;
}
.evolutions-wrap {
    border-top: 1px solid #d1d1d1;
    padding-top: 10px;
}
.evolution {
    padding: 40px 0;
}
</style>
