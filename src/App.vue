<template>
  <div class="buscar q-pa-md q-gutter-md">
    <div class="row q-col-gutter-md items-center">
      <div class="cajaBuscar">
        <q-input 
          clearable 
          filled 
          color="purple-12" 
          v-model="texto" 
          label="Busca el POKÉMON" 
          outlined
          dense
          class="q-my-md"
          :prepend-icon="searchIcon"
        />
      </div>
      <q-btn 
        color="purple" 
        @click="buscarPokemon" 
        label="Buscar"
        class="q-ml-md"
        icon="search"
      />
    </div>

    <q-card v-if="pokemon" :style="cardStyle" class="q-mt-md q-pa-md" style="max-width: 800px;">
      <q-card-section>
        <div class="row justify-center">
          <div class="col-12 text-center">
            <p class="nombrePokemon"><strong>{{ pokemon.nombre }}</strong></p>
          </div>
        </div>
        <div class="row">
          <div class="col-8">
            <p><strong>Estadísticas:</strong></p>
            <div class="row q-gutter-md">
              <div class="col-6" v-for="stat in stats" :key="stat.label">
                <div class="text-center">{{ stat.label }}: {{ formatValue(stat.value) }}%</div>
                <q-linear-progress 
                  :value="stat.value / 100" 
                  color="black"
                  size="1em"
                  track-color="grey-3"
                  animated
                />
              </div>
            </div>
          </div>
          <div class="col-4 relative-position">
            <div class="idPokemon">#{{ pokemon.id }}</div>
            <img :src="pokemon.imagen" alt="Imagen del Pokémon" class="pokemon-imagen">
            <div class="caracteristicas">
              <div class="caracteristicas_2">
                <p><strong>Altura:</strong> {{ pokemon.altura }} dm</p>
              </div>
              <div class="caracteristicas_2">
                <p><strong>Peso:</strong> {{ pokemon.peso }} kg</p>
              </div>
              <div class="caracteristicas_2">
                <p><strong>Tipo:</strong></p>
                <div class="tipos">
                  <span v-for="tipo in tipos" :key="tipo" :style="{ backgroundColor: tipoColorMap[tipo] }" class="tipo">{{ tipo }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </q-card-section>
    </q-card>

    <q-card v-else-if="buscando" class="q-mt-md" style="max-width: 800px;">
      <q-card-section class="q-pt-lg q-pb-lg">
        <q-spinner-gears size="50px" color="purple" />
        <p class="text-h6 q-mt-sm">Buscando...</p>
      </q-card-section>
    </q-card>

    <q-card v-else class="q-mt-md" style="max-width: 400px;">
      <q-card-section>
        <p class="text-h6">No se encontró ningún Pokémon con ese nombre.</p>
      </q-card-section>
    </q-card>
  </div>
</template>

<script setup>
import axios from "axios";
import { ref, computed } from "vue";

const tipoMap = {
  normal: "Normal",
  fighting: "Lucha",
  flying: "Volador",
  poison: "Veneno",
  ground: "Tierra",
  rock: "Roca",
  bug: "Bicho",
  ghost: "Fantasma",
  steel: "Acero",
  fire: "Fuego",
  water: "Agua",
  grass: "Planta",
  electric: "Eléctrico",
  psychic: "Psíquico",
  ice: "Hielo",
  dragon: "Dragón",
  dark: "Siniestro",
  fairy: "Hada",
  unknown: "Desconocido",
  shadow: "Sombra"
};

const tipoColorMap = {
  Normal: "#A8A77A",
  Lucha: "#C22E28",
  Volador: "#A98FF3",
  Veneno: "#A33EA1",
  Tierra: "#E2BF65",
  Roca: "#B6A136",
  Bicho: "#A6B91A",
  Fantasma: "#735797",
  Acero: "#B7B7CE",
  Fuego: "#EE8130",
  Agua: "#6390F0",
  Planta: "#7AC74C",
  Eléctrico: "#F7D02C",
  Psíquico: "#F95587",
  Hielo: "#96D9D6",
  Dragón: "#6F35FC",
  Siniestro: "#705746",
  Hada: "#D685AD",
  Desconocido: "#6B7280",
  Sombra: "#8C7A8B"
};

const texto = ref("");
const pokemon = ref(null);
const buscando = ref(false);
const searchIcon = ref("search");

const stats = ref([
  { label: "PS", value: 0 },
  { label: "Ataque", value: 0 },
  { label: "Defensa", value: 0 },
  { label: "Atq. Esp.", value: 0 },
  { label: "Def. Esp.", value: 0 },
  { label: "Velocidad", value: 0 }
]);

const tipos = ref([]);

function formatValue(value) {
  return value.toFixed(0);
}

const cardStyle = computed(() => {
  if (pokemon.value) {
    const tipos = pokemon.value.tipo.split(", ");
    if (tipos.length === 1) {
      return { backgroundColor: tipoColorMap[tipos[0]] || "#FFFFFF" };
    } else if (tipos.length === 2) {
      const color1 = tipoColorMap[tipos[0]] || "#FFFFFF";
      const color2 = tipoColorMap[tipos[1]] || "#FFFFFF";
      return { background: `linear-gradient(to right, ${color1}, ${color2})` };
    }
  }
  return { backgroundColor: "#FFFFFF" };
});

async function buscarPokemon() {
  if (!texto.value) {
    alert("Por favor, ingrese un nombre de Pokémon.");
    return;
  }

  buscando.value = true;
  pokemon.value = null;
  
  try {
    const response = await axios.get(`https://pokeapi.co/api/v2/pokemon/${texto.value.toLowerCase()}`);
    const pokemonData = response.data;
    
    const tipoEnEspañol = pokemonData.types.map(type => tipoMap[type.type.name]);
    tipos.value = tipoEnEspañol;
    
    const statsData = {};
    pokemonData.stats.forEach(stat => {
      statsData[stat.stat.name] = stat.base_stat;
    });

    stats.value = [
      { label: "PS", value: statsData.hp },
      { label: "Ataque", value: statsData.attack },
      { label: "Defensa", value: statsData.defense },
      { label: "Atq. Esp.", value: statsData["special-attack"] },
      { label: "Def. Esp.", value: statsData["special-defense"] },
      { label: "Velocidad", value: statsData.speed }
    ];

    pokemon.value = {
      imagen: pokemonData.sprites.other["official-artwork"].front_default,
      nombre: pokemonData.name,
      id: pokemonData.id,
      altura: pokemonData.height,
      peso: pokemonData.weight,
      tipo: tipoEnEspañol.join(", ")
    };
  } catch (error) {
    console.error("Error al buscar el Pokémon:", error);
    alert("No se encontró ningún Pokémon con ese nombre.");
  } finally {
    buscando.value = false;
  }
}
</script>

<style scoped>
.buscar {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
}

.cajaBuscar {
  display: flex;
  align-items: center;
  gap: 1rem;
  max-width: 400px;
}

.pokemon-imagen {
  max-width: 100%;
  position: relative;
  z-index: 1;
}

.relative-position {
  position: relative;
}

.idPokemon {
  font-size: 8rem;
  opacity: 0.3;
  position: absolute;
  top: 30%;
  left: -40%;
  transform: translate(-50%, -50%);
  z-index: 0;
}

.nombrePokemon {
  font-family: 'Press Start 2P', sans-serif;
  font-size: 2rem;
  margin-bottom: 1rem;
}

.col-8 {
  font-family: 'Press Start 2P', sans-serif;
  font-size: 16px;
}

.caracteristicas {
  font-family: 'Press Start 2P', sans-serif;
}

.tipo {
  display: inline-block;
  padding: 0.2rem 0.5rem;
  border: 2px solid black; /* Borde negro */
  border-radius: 4px;
  margin: 0.2rem;
}

@media (max-width: 600px) {
  .col-4, .col-8 {
    width: 100%;
  }
  
  .idPokemon {
    font-size: 4rem;
  }

  .nombrePokemon {
    font-size: 1.5rem;
  }
}
</style>
