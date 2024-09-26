<template>
  <div id="app">
    <img src="https://cdn.worldvectorlogo.com/logos/pokemon-23.svg" alt="Pokemon Logo" class="logo" />

    <form @submit.prevent="searchPokemon" class="search-form">
      <input type="text" v-model="pokemonName" autocomplete="off" placeholder="Busca un Pokémon" />
    </form>

    <div :class="['poke-card', { 'poke-card-expanded': !noResult }]">
      <div class="poke-card-content">
        <div class="img-container" :style="backgroundStyle">
          <img class="poke-img" :src="pokeImgSrc" alt="Pokémon" />
        </div>
        <div v-if="noResult" class="poke-message">{{ pokeMessage }}</div>
        <div v-else class="poke-info">
          <div class="poke-name">{{ pokeName }}</div>
          <div class="poke-id">{{ pokeIdText }}</div>
          <div class="poke-type-title">Tipo de Pokémon:</div>
          <div class="poke-types">
            <div v-for="type in pokeTypes" :key="type" :style="{ backgroundColor: typeColors[type] }">
              {{ type }}
            </div>
          </div>
          <div class="poke-weaknesses-title">Debilidades:</div>
          <div class="poke-weaknesses">
            <div v-for="weakness in pokeWeaknesses" :key="weakness" :style="{ backgroundColor: weaknessesMap[weakness] }">
              {{ weakness }}
            </div>
          </div>
        </div>
        <div v-if="!noResult" class="poke-stats">
          <div v-for="stat in pokeStats" :key="stat.stat.name" class="stat-item">
            <div class="stat-name">{{ stat.stat.name }}</div>
            <div class="progress-container">
              <div class="progress-bar" :style="{ width: `${(stat.base_stat / 225) * 100}%` }"></div>
              <div class="progress-label">{{ stat.base_stat }}/225</div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      pokemonName: '',
      pokeName: 'Pokedex',
      pokeIdText: '',
      pokeImgSrc: 'https://i.pinimg.com/originals/a6/4f/c7/a64fc73a5a257f7c6797205bd46d4842.png',
      pokeTypes: [],
      pokeWeaknesses: [],
      pokeStats: [],
      pokeMessage: '',
      noResult: true,
      typeColors: {
        electric: '#FFEA70',
        normal: '#B09398',
        fire: '#FF675C',
        water: '#0596C7',
        ice: '#AFEAFD',
        rock: '#999799',
        flying: '#7AE7C7',
        grass: '#4A9681',
        psychic: '#F272B6',
        bug: '#D4E157',
        poison: '#A040A0',
        ground: '#E1C699',
        ghost: '#6D28D9',
        dark: '#5A5A5A',
        dragon: '#DA627D',
        steel: '#1D8A99',
        fighting: '#2F2F2F',
        default: '#2A1A1F',
      },
      weaknessesMap: {},
    };
  },
  computed: {
    backgroundStyle() {
      const mainType = this.pokeTypes[0] || 'default';
      const bgColor = this.typeColors[mainType];
      return {
        background: `radial-gradient(${bgColor} 33%, transparent 33%), rgba(0, 0, 0, 0.1)`,
        backgroundSize: '10px 10px',
        borderRadius: '50%',
        padding: '15px',
      };
    },
  },
  mounted() {
    this.loadRandomPokemon();
  },
  methods: {
    searchPokemon() {
      fetch(`https://pokeapi.co/api/v2/pokemon/${this.pokemonName.toLowerCase()}`)
        .then((data) => data.json())
        .then((response) => {
          this.renderPokemonData(response);
          return response.types.map((typeInfo) => fetch(typeInfo.type.url));
        })
        .then((responses) => Promise.all(responses))
        .then((typeData) => Promise.all(typeData.map((res) => res.json())))
        .then((typeInfo) => this.renderPokemonWeaknesses(typeInfo))
        .catch(() => this.renderNotFound());
    },
    loadRandomPokemon() {
      const minId = 1;
      const maxId = 898;

      const randomId = Math.floor(Math.random() * (maxId - minId + 1)) + minId;

      fetch(`https://pokeapi.co/api/v2/pokemon/${randomId}`)
        .then((response) => response.json())
        .then((data) => {
          this.renderPokemonData(data);
          return data.types.map((typeInfo) => fetch(typeInfo.type.url));
        })
        .then((responses) => Promise.all(responses))
        .then((typeData) => Promise.all(typeData.map((res) => res.json())))
        .then((typeInfo) => this.renderPokemonWeaknesses(typeInfo))
        .catch(() => this.renderNotFound());
    },
    renderPokemonData(data) {
      this.pokeName = data.name.toUpperCase();
      this.pokeIdText = `ID: ${data.id}`;
      this.pokeImgSrc = data.sprites.front_default || 'https://i.pinimg.com/originals/a6/4f/c7/a64fc73a5a257f7c6797205bd46d4842.png';
      this.noResult = false;
      this.pokeTypes = data.types.map((type) => type.type.name);
      this.renderPokemonStats(data.stats);
      this.pokeMessage = '';
    },
    renderPokemonWeaknesses(typeData) {
      this.pokeWeaknesses = [];
      const weaknesses = new Set();

      typeData.forEach((type) => {
        type.damage_relations.double_damage_from.forEach((damage) => {
          weaknesses.add(damage.name);
          if (!this.weaknessesMap[damage.name]) {
            this.weaknessesMap[damage.name] = this.typeColors[damage.name] || this.typeColors.default;
          }
        });
      });

      this.pokeWeaknesses = Array.from(weaknesses);
    },
    renderPokemonStats(stats) {
      this.pokeStats = stats;
    },
    renderNotFound() {
      this.noResult = true;
      this.pokeImgSrc = 'https://i.pinimg.com/originals/a6/4f/c7/a64fc73a5a257f7c6797205bd46d4842.png';
      this.pokeMessage = 'Pokémon no encontrado.';
      this.pokeTypes = [];
      this.pokeWeaknesses = [];
      this.pokeStats = [];
    },
  },
};
</script>

<style>
body {
  background-image: url('https://images6.alphacoders.com/112/1121014.jpg');
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  margin: 0;
  padding: 0;
  font-family: 'Roboto Mono', monospace;
  box-sizing: border-box;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
}

#app {
  width: 100%;
  max-width: 500px;
  height: auto;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
}

.logo {
  width: 100%;
  max-width: 300px;
  margin-bottom: 10px;
}

.search-form {
  width: 100%;
  margin-bottom: 15px;
}

input {
  width: 100%;
  padding: 8px;
  border-radius: 4px;
  border: 1px solid #ccc;
}

.poke-card {
  position: relative;
  width: 100%;
  max-width: 400px;
  border-radius: 10px;
  color: #000;
  padding: 20px;
  background-color: #FFF;
  border: 2px solid #000;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  margin-bottom: 15px;
  transition: max-width 0.3s, min-height 0.3s;
}

.poke-card-content {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.poke-card-expanded {
  width: 300px;
  overflow: auto;
  padding: 10px;
}

.poke-card-expanded .img-container {
  width: 100%;
  height: 200px;
  margin-bottom: 10px;
  border-radius: 8px;
  overflow: hidden;
  background-color: #fff;
  display: flex;
  justify-content: center;
  align-items: center;
  box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.2);
}

.poke-img {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
}

.poke-info {
  text-align: center;
  width: 100%;
  margin-bottom: 15px;
}

.poke-name {
  font-size: 1.2em;
  font-weight: bold;
  margin-bottom: 10px;
}

.poke-id {
  font-size: 1em;
  color: #555;
}

.poke-type-title,
.poke-weaknesses-title {
  font-weight: bold;
  margin-top: 10px;
}

.poke-types,
.poke-weaknesses {
  display: flex;
  gap: 6px;
  justify-content: center;
  flex-wrap: wrap;
}

.poke-types div,
.poke-weaknesses div {
  padding: 4px 8px;
  margin: 3px;
  border-radius: 8px;
  border: 1px solid #ddd;
  text-transform: capitalize;
  color: #FFF;
  font-weight: bold;
}

.poke-stats {
  width: 100%;
  margin-top: 15px;
  padding: 8px;
  border-radius: 8px;
  background-color: #EFEFEF;
}

.stat-item {
  display: flex;
  flex-direction: column;
}

.stat-name {
  font-weight: bold;
  margin-bottom: 4px;
}

.progress-container {
  width: 100%;
  background-color: #e0e0e0;
  border-radius: 20px;
  height: 20px;
  margin-top: 4px;
  overflow: hidden;
  position: relative;
}

.progress-bar {
  height: 100%;
  background-color: #007bff;
  border-radius: 20px;
  transition: width 0.3s;
}

.progress-label {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #FFF;
  font-weight: bold;
  font-size: 14px;
  pointer-events: none;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.5);
}

.poke-message {
  font-size: 1.2em;
  color: #FF0000;
}

.no-result .poke-img {
  width: 80px;
  height: 80px;
}

@media (max-width: 768px) {
  .poke-card {
    padding: 8px;
  }

  .poke-card-expanded {
    width: 280px;
  }

  .poke-name {
    font-size: 1.3em;
  }

  .poke-id {
    font-size: 0.9em;
  }

  .progress-container {
    height: 18px;
  }

  .progress-label {
    font-size: 0.9em;
  }
}

@media (max-width: 480px) {
  .poke-card {
    padding: 4px;
  }

  .poke-card-expanded {
    width: 250px;
  }

  .poke-name {
    font-size: 1.2em;
  }

  .poke-id {
    font-size: 0.8em;
  }

  .progress-container {
    height: 15px;
  }

  .progress-label {
    font-size: 0.8em;
  }
}
</style>
