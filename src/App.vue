<template>
  <div id="app">
    <div v-if="viewType === 'options'">
      <p>Usatawienia łańcucha:</p>
      <label>
        Liczba słów:
        <input type="text" v-model="numberOfWords">
      </label>
      <label>
        Nazwa łańcucha:
        <input type="text" v-model="chainName">
      </label>
      <button @click="generate">Generuj</button>
    </div>

    <div v-if="viewType === 'results'">
      <div v-for="(word, i) in generatedWords" :key="i">
        {{ word.pl.toLowerCase() }}
        <img :src="word.img"/>
      </div>

    </div>
  </div>
</template>

<script>
import {words} from './helpers/words'

export default {
  name: 'App',
  components: {},
  data() {
    return {
      numberOfWords: 10,
      chainName: '',
      viewType: 'options', // options | results
      words: [],
      generatedWords: [],
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
  },
  computed: {},
  methods: {
    generate() {


      for (let i = 0; i < Number(this.numberOfWords); i++) {
        const randomIndex = Math.floor(Math.random() * (this.words.length));
        // dodac tak, żeby się nie powtarzało
        const word = this.words[randomIndex]

        const query =
            `key=20280075-0a41ab4fcc29805bbe5b60f99` +
            `&q=${word.en} isolated` +
            `&image_type=vector` + // "all", "photo", "illustration", "vector"
            `&safesearch=true` +
            `&per_page=3` + // 3 - 200
            `&image_type=illustration`

        console.log(query)
        const req = `https://pixabay.com/api/?${query}` // https://pixabay.com/api/docs/
        fetch(req)
            .then(resp => resp.json())
            .then((jsonResp) => {
              const imgUrl = jsonResp.hits[0] && jsonResp.hits[0].webformatURL ? jsonResp.hits[0].webformatURL : null
              if (imgUrl) word.img = imgUrl
              this.generatedWords.push(word)
            })
      }
      this.viewType = 'results'
    },
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
