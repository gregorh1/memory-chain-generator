<template>
  <div id="app">
    <div v-if="viewType === 'main'" style="margin-bottom: 60px">
      <h2>Generuj łańcuch</h2>
      <div>
        <label class="is-use-first-word">
          <input type="checkbox" v-model="isFirstElAsName" class="text-field" style="width:40px">
          Użyj pierwszego słowa jako nazwy
        </label>
        <div>~ albo ~</div>
        <label>
          Wpisz swoją nazwę:<br>
          <input type="text" v-model="chainName" class="text-field" :disabled="isFirstElAsName">
        </label>
      </div>
      <label style="display: block">
        Liczba słów (1-400):
        <input type="text" v-model="numberOfWords" class="text-field number">
      </label>
      <button @click="generate" class="pointer">Generuj</button>

      <h2 style="margin-top: 50px">Zapisane łańcuchy</h2>
      <div v-for="(chain, i) in myChains" :key="i" class="saved-chains-item">
        <span @click="() => showChain(chain.name)" style="margin-right: 20px;" class="pointer">
          {{ chain.name }} ({{ chain.chain.length }})
        </span>
        <span @click="() => removeChain(chain.name)" class="pointer">[usuń]</span>
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
      numberOfWords: 20,
      chainName: 'Mój łańcuch ',
      isFirstElAsName: true,
      viewType: 'main', // main | chain
      words: [],
      generatedChain: [],
      myChains: [],
    }
  },
  created() {
    this.words = [
      // ...words.anatomy,
      // ...words.characterTraits,
      // ...words.emotions,
      // ...words.homonimy,
      // ...words.cities,
      ...words.colors,
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
      const ifNameNotUniqueAddSuffix = (name) => {
        const isUnique = (otherName) => {
          let unique = true;
          storedChains.forEach((chain) => {
            if (chain.name === otherName) unique = false;
          })
          return unique
        }
        if (!storedChains || isUnique(name)) return name;

        let newName = '';
        const suffix = name.match(/\[(\d*)\]/);
        const addedNum = suffix ? Number(suffix[1]) : null;
        let nextNum = addedNum + 1;
        if (suffix) newName = name.replace(/\[(\d*)\]/, `[${nextNum}]`)
        else newName = `${name}[1]`;

        if (isUnique(newName)) return newName;
        else {
          while (!isUnique(newName)) {
            nextNum++;
            newName = newName.replace(/\[(\d*)\]/, `[${nextNum}]`)
          }
          return newName;
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

      if (this.isFirstElAsName) this.chainName = this.generatedChain[0].toUpperCase()
      this.chainName = ifNameNotUniqueAddSuffix(this.chainName);

      const chainObj = {
        name: this.chainName,
        chain: this.generatedChain,
      }
      if (storedChains) {
        storedChains.unshift(chainObj)
        localStorage.setItem('myChains', JSON.stringify(storedChains))
        this.myChains = storedChains;
      } else {
        localStorage.setItem('myChains', JSON.stringify([chainObj]))
        this.myChains = [chainObj];
      }
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
      const updatedMyChains = this.myChains.filter((chain) => {
        return chain.name !== chainName
      })
      this.myChains = updatedMyChains
      localStorage.setItem('myChains', JSON.stringify(updatedMyChains))
    },
    chandleBackButton() {
      this.viewType = 'main';
      this.generatedChain = [];
      this.chainName = '';
    },
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  color: #2c3e50;
  max-width: 600px;
  margin: 0 auto;
}

label {
  display: block;
  margin: 10px;
}

.text-field {
  border: 0;
  border-bottom: solid 1px #d2d2d2;
  width: 100%;
  max-width: 500px;
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
  background: #0075ff;
  border: solid 1px #0075ff;
  border-radius: 5px;
  font: inherit;
  color: white;
}

hr {
  border: 1px solid #d2d2d2;
  margin: 30px 0;
}

.saved-chains-item span {
  display: inline-block;
  padding: 10px;
  cursor: pointer;
}

.pointer {
  cursor: pointer;

}

.pointer:hover {
  filter: drop-shadow(0px 0px 1px lightgrey);
}

.is-use-first-word {
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
