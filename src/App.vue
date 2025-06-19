<template>
  <div class="app">

    <!-- Header -->
    <header>
      <div class="logo">
        <img src="./assets/logo.png" alt="logo" />
        <h1>orzArchives</h1>
      </div>

      <input v-model="search" placeholder="Rechercher" />
      <button @click="toggleFilters">Filtres</button>
    </header>

    <!-- Filtres -->
    <div class="filters" v-if="showFilters">
      <select v-model="selectedCategory">
        <option value="">Toutes les catégories</option>
        <option v-for="cat in categories" :key="cat" :value="cat">{{ cat }}</option>
      </select>

      <select v-model="selectedCountry">
        <option value="">Tous les pays</option>
        <option v-for="countryName in countryNamesArray" :key="countryName" :value="countryCodes[countryName]">
          {{ countryName }}
        </option>
      </select>


      <!-- Nouveau filtre motorisation -->
      <select v-model="selectedMotorisation">
        <option value="">Toutes motorisations</option>
        <option v-for="motor in motorisations" :key="motor" :value="motor">{{ motor }}</option>
      </select>

      <!-- Sélecteur tri -->
      <select v-model="sortKey">
        <option value="">Trier par</option>
        <option value="annee">Année</option>
        <option value="puissance">Puissance</option>
        <option value="poids">Poids</option>
        <option value="prix">Prix</option>
      </select>

      <select v-model="sortOrder" :disabled="!sortKey">
        <option value="asc">Ascendant</option>
        <option value="desc">Descendant</option>
      </select>
    </div>

    <!-- Car Grid-->
    <div class="car-grid">
      <div class="car-card" v-for="car in filteredCars" :key="car.ID" @click="selectCar(car)">
        <img :src="car.thumbnail" />
        <h3> {{ car.fields.annee }} {{ car.fields.marque }}</h3>
        <h4>{{ car.fields.modele }}</h4>
        <p>{{ car.categories[0] }}</p>
      </div>
    </div>

    <!-- Car Modal -->
    <div v-if="selectedCar" class="modal-overlay" @click.self="selectedCar = null">
      <div class="modal-content">
        <button class="close-btn" @click="selectedCar = null">×</button>
        <img :src="selectedCar.thumbnail" />
        <br>
        <img :src="`https://flagcdn.com/w40/${selectedCar.pays}.png`" alt="Drapeau" class="drapeau">
        <h2>{{ selectedCar.fields.annee }} {{ selectedCar.fields.marque }} </h2>
        <h3>{{ selectedCar.fields.modele }}</h3>
        <p>{{ selectedCar.categories[0] }}</p>
        <div class="specs">
          <div><strong>Puissance</strong> : {{ formatNumber(selectedCar.fields.puissance) }} ch</div>
          <div><strong>Poids</strong> : {{ formatNumber(selectedCar.fields.poids) }} kg</div>
          <div><strong>Motorisation</strong> : {{ selectedCar.fields.motorisation }}</div>
          <div><strong>Prix</strong> : {{ formatNumber(selectedCar.fields.prix) }} Cr</div>
        </div>

      </div>
    </div>

  </div>
</template>

<script>
import { onMounted, ref, computed } from 'vue'


export default {
  setup() {
    const cars = ref([])
    const selectedCar = ref(null)
    const search = ref('')
    const selectedCategory = ref('')
    const selectedCountry = ref('')
    const selectedMotorisation = ref('')
    const showFilters = ref(false)

    const sortKey = ref('')
    const sortOrder = ref('asc')

    const categories = ref([])
    const countries = ref([])
    const motorisations = ref([])

    const fetchCars = async () => {
      const res = await fetch('https://sae401-25.mmi-stdie.fr/charlyd/wp-json/wp/v2/voiture')
      const data = await res.json()
      cars.value = data

      categories.value = [...new Set(data.flatMap(car => car.categories))]
      countries.value = [...new Set(data.flatMap(car => car.pays))]
      motorisations.value = [...new Set(data.map(car => car.fields.motorisation).filter(Boolean))]
    }

    // Liste des noms complets
    const countryNamesArray = [
      'France',
      'États-Unis',
      'Allemagne',
      'Italie',
      'Espagne',
      'Japon',
      'Royaume-Uni',
      // ajoute d'autres noms ici
    ]

    // Mapping nom complet => code pays
    const countryCodes = {
      'France': 'fr',
      'États-Unis': 'us',
      'Allemagne': 'de',
      'Italie': 'it',
      'Espagne': 'es',
      'Japon': 'jp',
      'Royaume-Uni': 'gb',
      // ajoute d'autres mappings ici
    }

    const filteredCars = computed(() => {
      let filtered = cars.value.filter(car => {
        const matchesSearch = car.title.toLowerCase().includes(search.value.toLowerCase())
        const matchesCategory = selectedCategory.value ? car.categories.includes(selectedCategory.value) : true
        const matchesCountry = selectedCountry.value ? car.pays.includes(selectedCountry.value) : true
        const matchesMotorisation = selectedMotorisation.value ? car.fields.motorisation === selectedMotorisation.value : true
        return matchesSearch && matchesCategory && matchesCountry && matchesMotorisation
      })

      if (sortKey.value) {
        filtered = filtered.slice().sort((a, b) => {
          const valA = Number(a.fields[sortKey.value])
          const valB = Number(b.fields[sortKey.value])
          if (isNaN(valA)) return 1
          if (isNaN(valB)) return -1
          return sortOrder.value === 'asc' ? valA - valB : valB - valA
        })
      }

      return filtered
    })

    const toggleFilters = () => {
      showFilters.value = !showFilters.value
    }

    const selectCar = (car) => {
      selectedCar.value = car
    }

    const formatNumber = (val) => {
      const num = Number(val)
      if (isNaN(num)) return val
      return new Intl.NumberFormat('fr-FR').format(num)
    }

    onMounted(fetchCars)

    return {
      cars,
      selectedCar,
      selectCar,
      search,
      selectedCategory,
      selectedCountry,
      selectedMotorisation,
      filteredCars,
      showFilters,
      toggleFilters,
      categories,
      countries,
      motorisations,
      sortKey,
      sortOrder,
      formatNumber,
      countryNamesArray,
      countryCodes,
    }
  }

}
</script>
