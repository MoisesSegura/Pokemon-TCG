<template>
  <div class="flex flex-col items-center mb-6">
    <img
      src="/images/pokemonLogo.png"
      alt="Pokémon TCG Logo"
      class="w-150 mb-4"
    />

    <div class="flex flex-col relative">
      <div class="flex items-center gap-2">
        <input
          v-model="nombre"
          @input="filtrarSugerencias"
          @keydown.down.prevent="moverSeleccion(1)"
          @keydown.up.prevent="moverSeleccion(-1)"
          @keydown.enter.prevent="seleccionarConEnter"
          type="text"
          placeholder="Search by name..."
          class="px-6 py-2 rounded-lg font-bold text-white 
           border-4 border-[#3463b0] 
           bg-gradient-to-b from-[#ee171f] to-red-800 
           shadow-md hover:shadow-lg 
           hover:from-red-600 hover:to-red-900 
           focus:outline-none focus:ring-2 focus:ring-[#3463b0]"
        />
        <button
          @click="buscar"
          class="px-6 py-2 rounded-lg font-bold text-white 
           border-4 border-[#3463b0] 
           bg-gradient-to-b from-[#ee171f] to-red-800 
           shadow-md hover:shadow-lg 
           hover:from-red-600 hover:to-red-900 
           transition-transform transform hover:scale-105
           focus:outline-none focus:ring-2 focus:ring-[#3463b0]"
        >
          Search
        </button>
      </div>

      <!-- Lista de sugerencias -->
      <ul
        v-if="sugerencias.length > 0"
        class="absolute top-full mt-2 w-full bg-white border border-gray-300 rounded-lg shadow-md z-10"
      >
        <li
          v-for="(p, index) in sugerencias"
          :key="p"
          @click="seleccionarSugerencia(p)"
          :class="[
            'px-4 py-2 cursor-pointer',
            index === indiceSeleccionado ? 'bg-[#3463b0] text-white' : 'hover:bg-gray-200'
          ]"
        >
          {{ p }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      nombre: "",
      allPokemon: [],
      sugerencias: [],
      indiceSeleccionado: -1,
    };
  },
  mounted() {
    fetch("/pokemon_names.json")
      .then(res => res.json())
      .then(data => {
        this.allPokemon = data;
      })
      .catch(err => console.error("Error cargando JSON:", err));
  },
  methods: {
    buscar() {
      this.$emit("buscarPokemon", this.nombre.trim());
    },
    filtrarSugerencias() {
      if (this.nombre.length > 0) {
        const query = this.nombre.toLowerCase();
        this.sugerencias = this.allPokemon
          .filter(p => p.toLowerCase().includes(query))
          .slice(0, 5);
        this.indiceSeleccionado = -1;
      } else {
        this.sugerencias = [];
      }
    },
    moverSeleccion(direccion) {
      if (this.sugerencias.length === 0) return;
      this.indiceSeleccionado =
        (this.indiceSeleccionado + direccion + this.sugerencias.length) %
        this.sugerencias.length;
    },
    seleccionarConEnter() {
      if (
        this.indiceSeleccionado >= 0 &&
        this.indiceSeleccionado < this.sugerencias.length
      ) {
        this.seleccionarSugerencia(this.sugerencias[this.indiceSeleccionado]);
      } else {
        this.buscar();
      }
    },
    seleccionarSugerencia(nombre) {
      this.nombre = nombre;
      this.sugerencias = [];
      this.indiceSeleccionado = -1;
      this.buscar();
    },
  },
};
</script>
