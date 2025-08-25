<template>
  <div :class="['min-h-screen p-30 transition-colors duration-500', fondoPorTipo]">
    <BuscadorPokemon @buscarPokemon="buscarPorNombre" />

    <div v-if="loading" class="flex flex-col items-center justify-center py-10">
      <img
        src="https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/items/poke-ball.png"
        alt="Pokeball loader"
        class="w-16 h-16 animate-spin"
      />
      <p class="mt-4 text-lg text-gray-700 font-semibold">Loading cards...</p>
    </div>

    <div v-else-if="filteredCards.length === 0">No cards found.</div>
    <div
      v-else
      class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-5 gap-4"
    >
      <PokemonCard
        v-for="card in filteredCards"
        :key="card.id"
        :card="card"
        @ver-info="mostrarDetalle"
      />
    </div>

    <div
      v-if="cartaSeleccionada"
      class="fixed inset-0 flex items-center justify-center z-50"
      style="perspective: 1000px;"
    >
      <div
        class="absolute inset-0 bg-black opacity-50"
        @click="cartaSeleccionada = null"
      ></div>

      <div
        ref="cartaTilt"
        @mousemove="moverCarta"
        @mouseleave="resetearCarta"
        class="relative rounded-lg shadow-lg p-2 max-w-md w-full z-10 transition-transform duration-300 ease-out"
        :style="{ transform: tiltTransform, willChange: 'transform', transformStyle: 'preserve-3d' }"
      >
        <div
          class="absolute inset-0 pointer-events-none transition-opacity duration-300"
          :style="lightStyle"
        ></div>
        <img :src="cartaSeleccionada.images?.large" alt="" class="w-full" />
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import BuscadorPokemon from "./components/BuscadorPokemon.vue";
import PokemonCard from "./components/PokemonCard.vue";

export default {
  components: { PokemonCard, BuscadorPokemon },
  data() {
    return {
      cards: [],
      filteredCards: [],
      loading: true,
      cartaSeleccionada: null,
      tiltTransform: "scale(1)",
      lightStyle: {},
    };
  },
  methods: {
    mostrarDetalle(card) {
      this.cartaSeleccionada = card;
    },
    moverCarta(e) {
      const card = this.$refs.cartaTilt;
      const rect = card.getBoundingClientRect();
      const x = e.clientX - rect.left;
      const y = e.clientY - rect.top;
      const centerX = rect.width / 2;
      const centerY = rect.height / 2;
      const rotateX = ((y - centerY) / centerY) * -10;
      const rotateY = ((x - centerX) / centerX) * 10;

      this.tiltTransform = `rotateX(${rotateX}deg) rotateY(${rotateY}deg) scale(1.02)`;

      const percentX = (x / rect.width) * 100;
      const percentY = (y / rect.height) * 100;

      this.lightStyle = {
        backgroundImage: `radial-gradient(circle at ${percentX}% ${percentY}%, rgba(255,255,255,0.3), transparent 60%)`,
      };
    },
    resetearCarta() {
      this.tiltTransform = "rotateX(0deg) rotateY(0deg) scale(1)";
      this.lightStyle = {};
    },
    async buscarPorNombre(nombre) {
      this.loading = true;
      try {
        const response = await axios.get(
          `https://api.pokemontcg.io/v2/cards?q=name:${encodeURIComponent(
            nombre
          )}`,
          {
            headers: {
              "X-Api-Key": import.meta.env.VITE_POKEMON_API_KEY,
            },
          }
        );

        this.filteredCards = response.data.data || [];
      } catch (error) {
        console.error("Error al buscar:", error);
      } finally {
        this.loading = false;
      }
    },
  },

  async mounted() {
    try {
      const response = await axios.get(
        "https://api.pokemontcg.io/v2/cards?page=1&pageSize=250",
        {
          headers: {
            "X-Api-Key": import.meta.env.VITE_POKEMON_API_KEY,
          },
        }
      );

      this.cards = response.data.data;
      this.filteredCards = this.cards;

      this.loading = false;
    } catch (error) {
      console.error("Error al cargar cartas:", error);
      this.loading = false;
    }
  },

  computed: {
    fondoPorTipo() {
      if (!this.filteredCards.length) {
        return "bg-gradient-to-r from-red-300 via-green-300 to-blue-300";
      }
      const tipo = this.filteredCards[0].types?.[0] || "default";
      switch (tipo) {
        case "Fire":
          return "bg-gradient-to-br from-red-400 to-yellow-300";
        case "Water":
          return "bg-gradient-to-br from-blue-400 to-cyan-300";
        case "Grass":
          return "bg-gradient-to-br from-green-400 to-lime-300";
        case "Lightning":
          return "bg-gradient-to-br from-yellow-300 to-yellow-100";
        case "Psychic":
          return "bg-gradient-to-br from-purple-400 to-pink-300";
        case "Ground":
          return "bg-gradient-to-br from-yellow-800 to-yellow-400";
        case "Metal":
          return "bg-gradient-to-br from-gray-700 to-gray-400";
        case "Fighting":
          return "bg-gradient-to-br from-red-800 to-red-200";
        case "Darkness":
          return "bg-gradient-to-r from-slate-900 to-slate-700";
        case "Dragon":
          return "bg-gradient-to-br from-blue-300 to-indigo-200";
        case "Fairy":
          return "bg-gradient-to-br from-pink-300 to-pink-100";
        case "Colorless":
          return "bg-gradient-to-br from-white to-grey-100";
        default:
          return "bg-gradient-to-r from-red-300 via-green-300 to-blue-300";
      }
    },
  },
};
</script>
