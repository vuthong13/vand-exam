<template>
  <div class="pokemon">
    <div class="container">
      <h1>
        <img src="@/assets/images/pokemon-logo.png" alt="">
      </h1>
      <div class="filter-wrapper">
        <p>Pokemon type filter:</p>
        <select v-model="typeFilter" @change="updateList()">
          <option value="" selected>None</option>
          <option v-for="(item, i) in pokemonType" :value="i" :key="i">{{ item }}</option>
        </select>
      </div>
      <div class="table-wrapper">
        <table class="pokemon-list">
          <thead>
            <tr>
              <th v-for="(item, i) in attributeList" :key="i" @click="updateSort(item)" :class="[sort == item ? '' : '', sort == item && sortAsc ? '' : 'desc']">{{ item }}</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(item, i) in sortedPokemonList" :key="i">
              <td :class="[sort == propertyName ? 'active' : '', value != null ? propertyName : '']" v-for="(value, propertyName, index) in item" :key="index"  @click="selectPokemon(i)">
                <span v-if="propertyName.indexOf('type') > -1">{{ pokemonType[value] }}</span>
                <span v-else-if="propertyName == 'legendary'">{{ value ? 'Yes' : 'No' }}</span>
                <span v-else>{{ value }}</span>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      <div class="modal" v-show="showPopup">
        <div class="modal-bg" @click="showPopup = false"></div>
        <div class="modal-content">
          <div class="close" @click="showPopup = false"></div>
          <div class="modal-content-info">
            <div :class="[imgSrc ? '' : 'hide', 'modal-content-info__img']">
              <img :src="imgSrc" alt="">
            </div>
            <div class="modal-content-info__data">
              <ul>
                <li v-for="(value, propertyName, index) in pokemonList[selectedPokemon]" :key="index">
                  <p v-if="propertyName.indexOf('type') > -1"><b>{{ propertyName }}:</b> {{ pokemonType[value] }}</p>
                  <p v-else><b>{{ propertyName }}:</b> {{ value }}</p>
                </li>
              </ul>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      pokemonList: [],
      pokemonType: [],
      attributeList: [],
      sort: 'number',
      sortAsc: 1,
      typeFilter: '',
      imgSrc: '',
      selectedPokemon: 1,
      showPopup: false
    }
  },
  methods: {
    updateSort(i) {
      if (i != this.sort) {
        this.sort = i;
        this.sortAsc = 1;
      }
      else
       this.sortAsc = !this.sortAsc; 
    },
    selectPokemon(i) {
      this.selectedPokemon = i;
      this.imgSrc = '';
      this.getPokemonImage(this.pokemonList[i].id);
      this.showPopup = true;
    },
    getPokemonImage(id) {
      this.axios.get('https://api.eien-development.com/api/pokemon-api/pokemons/' + id + '/sprite', {
        responseType: 'blob'
      }).then((response) => {
        console.log(response.data);
        this.imgSrc = URL.createObjectURL(response.data);
      })
    },
    getData() {
      this.axios.get('https://api.eien-development.com/api/pokemon-api/pokemons').then((response) => {
        // console.log(response.data);
        this.pokemonList = response.data.data;
        this.attributeList = Object.keys(this.pokemonList[0]);
      })
      this.axios.get('https://api.eien-development.com/api/pokemon-api/types').then((response) => {
        this.pokemonType = response.data.data.map(item => item.name);
        // console.log(this.pokemonType);
      })
    },
    updateList() {
      if (this.typeFilter == '')
        this.axios.get('https://api.eien-development.com/api/pokemon-api/pokemons').then((response) => {
          // console.log(response.data);
          this.pokemonList = response.data.data;
        })
      else {
        this.axios.get('https://api.eien-development.com/api/pokemon-api/pokemons?filter[type]=' + this.typeFilter).then((response) => {
          // console.log(response.data);
          this.pokemonList = response.data.data;
        })
      }
    }
  },
  async beforeMount() {
    await this.getData();
  },
  computed: {
    sortedPokemonList() {
      if (this.sort) {
        if (this.sortAsc)
          return this.pokemonList.slice(0).sort((a,b) => (a[this.sort] > b[this.sort]) ? 1 : -1);
        else
          return this.pokemonList.slice(0).sort((a,b) => (a[this.sort] < b[this.sort]) ? 1 : -1);
      }
      else
        return this.pokemonList.slice(0);
    }
  }
}
</script>

<style lang="scss" scoped src="@/assets/css/styles.scss"></style>
