<template>
  <div id="app">
    <div v-if="viewType === 'main'" style="margin-bottom: 60px">
      <h2>Generuj łańcuch</h2>
      <label style="display: block">
        Nazwa:<br>
        <input type="text" v-model="chainName" class="text-field">
      </label>
      <label style="display: block">
        Liczba słów (1-400):
        <input type="text" v-model="numberOfWords" class="text-field number">
      </label>
      <button @click="generate">Generuj</button>

      <h2 style="margin-top: 50px">Zapisane łańcuchy</h2>
      <div v-for="(chain, i) in myChains" :key="i" class="saved-chains-item">
        <span @click="() => showChain(chain.name)" style="margin-right: 20px;">{{
            chain.name
          }} ({{ chain.chain.length }})</span>
        <span @click="() => removeChain(chain.name)">[usuń]</span>
      </div>
    </div>

    <show-chain v-else-if="viewType === 'chain'"
                @back="chandleBackButton"
                :name="chainName"
                :chain="generatedChain"/>
  </div>
</template>

<script>
import {words} from './helpers/words'
import ShowChain from "@/components/ShowChain";

export default {
  name: 'App',
  components: {
    ShowChain
  },
  data() {
    return {
      numberOfWords: 4,
      chainName: 'Łańcuch ',
      viewType: 'main', // main | chain
      words: [],
      generatedChain: [],
      myChains: [],
    }
  },
  created() {
    this.words = [
      ...words.anatomy,
      ...words.characterTraits,
      ...words.emotions,
      ...words.homonimy,
      ...words.colors,
      ...words.cities,
      ...words.places,
      ...words.fruits,
      ...words.vegetables,
      ...words.weather,
      ...words.nouns,
      ...words.sport,
      ...words.occupations,
      ...words.animals,
    ];
    this.myChains = JSON.parse(localStorage.getItem('myChains'))
  },
  computed: {},
  methods: {
    generate() {
      if (this.chainName === '' || this.numberOfWords < 1 || this.numberOfWords > 400) {
        alert('Nazwa nie może być pusta, liczba słów musi mieścić się wprzedziale od 1 do 400');
        return;
      }
      const storedChains = JSON.parse(localStorage.getItem('myChains'));
      if (storedChains) {
        let isNameUnique = true;
        storedChains.forEach((chain) => {
          if (chain.name === this.chainName) isNameUnique = false;
        })
        if (!isNameUnique) {
          alert('Taka nazwa już istnieje');
          return;
        }
      }
      const getUniqueRandomIndex = (times) => {
        if (times > 0) {
          const randomIndex = Math.floor(Math.random() * (this.words.length));
          const word = this.words[randomIndex];
          if (this.generatedChain.indexOf(word) === -1) {
            this.generatedChain.push(word);
            getUniqueRandomIndex(--times);
          } else getUniqueRandomIndex(times);
        }
      }
      getUniqueRandomIndex(this.numberOfWords);
      const chainObj = {
        name: this.chainName,
        chain: this.generatedChain,
      }
      if (storedChains) {
        storedChains.unshift(chainObj)
        localStorage.setItem('myChains', JSON.stringify(storedChains))
      } else {
        localStorage.setItem('myChains', JSON.stringify([chainObj]))
      }
      this.myChains = storedChains;
      this.viewType = 'chain';
    },
    showChain(chainName) {
      const chosenChain = this.myChains.find((chain) => {
        return chain.name === chainName;
      })
      this.generatedChain = chosenChain.chain;
      this.chainName = chosenChain.name
      this.viewType = 'chain'
    },
    removeChain(chainName) {
      console.log(chainName)
      const updatedMyChains = this.myChains.filter((chain) => {
        return chain.name !== chainName
      })
      this.myChains = updatedMyChains
      localStorage.setItem('myChains', JSON.stringify(updatedMyChains))
    },
    chandleBackButton() {
      this.viewType = 'main';
      this.generatedChain = [];
    },
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  color: #2c3e50;
}

label {
  display: block;
  margin: 20px;
}

.text-field {
  border: 0;
  border-bottom: solid 1px #d2d2d2;
  width: 100%;
  text-align: center;
  height: 30px;
  font: inherit;
}

.text-field:focus {
  outline: 0;
  border-color: black;
}

.text-field.number {
  width: 50px;
}

button {
  padding: 10px 50px;
  background: antiquewhite;
  border: solid 1px #c3b39e;
  border-radius: 5px;
  font: inherit;
}

hr {
  border: 1px solid #d2d2d2;
  margin: 30px 0;
}

.saved-chains-item {
  padding: 10px;
}
</style>
