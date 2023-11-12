<template>
  <div>
    <h1 class="page-header">Countries Information</h1>
    <div class="container">
      <div class="input-container">
        <div class="search-container">
          <input class="search-input" type="text" v-model="searchQuery" placeholder="Search country..." />
        </div>
        <select v-model="sortOrder" class="sort-select">
          <option value="" disabled>Select Sort Order</option>
          <option value="asc">Ascend</option>
          <option value="desc">Descend</option>
        </select>
      </div>
      <div class="pagination-container">
        <button class="pagination-btn" @click="previousPage" :disabled="currentPage === 1">Previous</button>
        <button class="pagination-btn" @click="nextPage" :disabled="currentPage === totalPages">Next</button>
      </div>

      <div class="country-container">
        <div class="country-card" v-for="country in paginatedCountries" :key="country.cca3">
          <div class="card-content">
            <img :src="country.flags.png" class="country-flag" @click="openModal(country)"/>
            <div class="country-details">
              <div class="country-name" @click="openModal(country)">{{ country.name.official }}</div>
              <div class="country-native">
                Code: {{ country.cca2 }}
              </div>
            </div>
          </div>
        </div>
      </div>

      <div class="modal-overlay" v-if="showModal">
        <div class="modal">
          <div class="modal-content">
            <div class="modal-country-image">
              <img :src="activeCountry.flags.png" class="modal-country-flag" />
            </div>
            <div class="modal-country-details">
              <div class="modal-country-info">
                <strong>{{ activeCountry.name.official }}</strong>
              </div>
              <div class="modal-info">
                <strong>CCA2:</strong> {{ activeCountry.cca2 }}
              </div>
              <div class="modal-info">
                <strong>CCA3:</strong> {{ activeCountry.cca3 }}
              </div>
              <div class="modal-info">
                <strong>Native Name:</strong> {{ getNativeName(activeCountry.name.nativeName) }}
              </div>
              <div class="modal-info">
                <strong>Alternative Name:</strong> {{ activeCountry.altSpellings.join(', ') }}
              </div>
              <div class="modal-info">
                <strong>Calling Codes:</strong> {{ activeCountry.idd.root }}
              </div>
            </div>
            <button class="close-button" @click="closeModal">Close</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import Fuse from 'fuse.js';

export default {
  data() {
    return {
      countries: [],
      sortOrder: null,
      searchQuery: '',
      currentPage: 1,
      cardsPerPage: 25,
      showModal: false,
      activeCountry: null,
    };
  },
  mounted() {
    this.fetchCountries();
  },
  computed: {
    filteredCountries() {
      let filtered = this.countries;

      if (this.searchQuery) {
        const options = {
          keys: ['name.official'],
          includeScore: true,
          findAllMatches: true,
          threshold: 0.4, // Adjust the threshold as needed
        };
        const fuse = new Fuse(filtered, options);
        filtered = fuse.search(this.searchQuery).map((result) => result.item);
      }

      if (this.sortOrder === 'asc') {
        return filtered.sort((a, b) => a.name.official.localeCompare(b.name.official));
      } else if (this.sortOrder === 'desc') {
        return filtered.sort((a, b) => b.name.official.localeCompare(a.name.official));
      } else {
        return filtered;
      }
    },
    totalPages() {
      return Math.ceil(this.filteredCountries.length / this.cardsPerPage);
    },
    paginatedCountries() {
      const startIndex = (this.currentPage - 1) * this.cardsPerPage;
      const endIndex = startIndex + this.cardsPerPage;
      return this.filteredCountries.slice(startIndex, endIndex);
    },
  },
  methods: {
    async fetchCountries() {
      try {
        const response = await axios.get('https://restcountries.com/v3.1/all');
        this.countries = response.data;
      } catch (error) {
        console.error('Error fetching countries:', error);
      }
    },
    getNativeName(nativeName) {
      if (typeof nativeName === 'object') {
        const languageCode = Object.keys(nativeName)[0];
        if (languageCode && nativeName[languageCode].official) {
          return nativeName[languageCode].official;
        }
      }
      return nativeName;
    },
    previousPage() {
      if (this.currentPage > 1) {
        this.currentPage--;
      }
    },
    nextPage() {
      if (this.currentPage < this.totalPages) {
        this.currentPage++;
      }
    },
    openModal(country) {
      console.log('Modal opened for:', country);
      this.activeCountry = country;
      this.showModal = true;
    },
    closeModal() {
      this.activeCountry = null;
      this.showModal = false;
    },
  },
};
</script>

<style scoped>
.page-header {
  font-family: Georgia, sans-serif;
  font-size: 45px;
  letter-spacing: -2px;
  text-align: center;
  
}

.input-container {
  display: flex;
  justify-content: space-between;
}

.search-container {
  flex-grow: 1;
}

.search-input,
.sort-select {
  height: 40px; 
}

.search-input {
  width: 100%;
  padding: 10px;
  font-size: 18px;
  border: 1px solid #ccc;
  border-radius: 5px;
  box-sizing: border-box;
  margin-bottom: 10px;
}

.sort-select {
  width: 150px;
  padding: 10px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 5px;
  box-sizing: border-box;
  margin-left: 10px;
}

.container {
  max-width: 95%;
  margin: 0 auto;
  padding: 20px;
}

.country-container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
  padding: 15px;
  margin: 0;
}

.country-card {
  background-color: #E0F2FE;
  border: 3px solid #808080;
  border-radius: 8px;
  width: 250px;
  height: 225px;
  margin: 0;
}

.card-content {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.country-flag {
  width: 100%;
  height: 125px;
  object-fit: cover;
  cursor: pointer;
}

.country-details {
  padding: 15px;
  display: flex;
  flex-direction: column;
  align-items: center; 
  text-align: center; 
}

.country-name {
  font-family: Georgia, sans-serif;
  font-weight: bold;
  font-size: 16px;
  cursor: pointer;
}

.country-native {
  margin-top: 10px;
}

.pagination-container {
  display: flex;
  justify-content: center;
  margin-top: 20px;
}

.pagination-btn {
  background-color: #3498db; 
  color: #ffffff; 
  border: none;
  padding: 10px 15px;
  margin-right: 15px;
  border-radius: 5px;
  cursor: pointer;
  font-size: 16px;
  transition: background-color 0.3s;
}

.pagination-btn:disabled {
  background-color: #bdc3c7; 
  cursor: not-allowed;
}

.pagination-btn:hover:not(:disabled) {
  background-color: #2980b9; 
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: rgba(0, 0, 0, 0.5);
}

.modal {
  background-color: #fff;
  border-radius: 10px;
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.2);
  width: 70vw;
  max-width: 400px;
  overflow: hidden;
}

.modal-content {
  display: flex;
  flex-direction: column;
}

.modal-country-image {
  text-align: center;
}

.modal-country-flag {
  width: 350px; 
  height: 250px; 
  padding: 20px;
  border-radius: 5px;
  object-fit: cover;
}

.modal-country-details {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}

.modal-country-info {
  font-family: Georgia, sans-serif;
  font-size: 20px;
  margin-bottom: 15px;
  text-indent: 0; 
  padding-left: 25px; 
}

.modal-info {
  margin-bottom: 5px;
  text-indent: 0; 
  padding-left: 25px; 
}

.close-button {
  background-color: #d9534f;
  color: #fff;
  border: none;
  padding: 10px 15px;
  cursor: pointer;
  font-size: 16px;
  transition: background-color 0.3s;
  display: inline-block; 
  margin-top: 20px;
}

.close-button:hover {
  background-color: #c9302c; 
}
</style>
