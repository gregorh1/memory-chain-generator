<template>
  <div style="margin-bottom: 100px">
    <div class="top-bar">
      <div @click="$emit('back')" class="back-btn pointer">&lt; wróć</div>
      <h2>{{ name }}</h2>
    </div>
    <div v-if="viewType === 'single'" class="container chain-element-container">
      <div class="chain-element">
        {{ currentElement.toUpperCase() }}
      </div>
      <div class="pointer" @click="nextElement">
        >
      </div>
    </div>
    <div v-else class="words-container">
      <div v-for="(word, i) in chain" :key="i" class="list-element pointer">
        <label>
          {{ i + 1 }}.
          <input type="checkbox" class="toggle" hidden/>
          <span class="word">
            {{ word.toUpperCase() }}
          </span>
        </label>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "show-chain",
  props: {
    chain: Array,
    name: String,
  },
  data() {
    return {
      viewType: 'single',
      currentElement: '',
    }
  },
  mounted() {
    this.currentElement = this.chain[0];
  },
  methods: {
    nextElement() {
      const currentElIndex = this.chain.indexOf(this.currentElement)
      if (currentElIndex <= this.chain.length - 2) {
        this.currentElement = this.chain[currentElIndex + 1]
      } else {
        this.currentElement = this.chain[0]
        this.viewType = 'list';
      }
    }
  },
}
</script>

<style scoped>

.container {
  display: flex;
  flex-direction: column;
  justify-content: space-evenly;
}

.chain-element-container {
  height: calc(100vh - 100px);
  font-size: 50px;
}

.chain-element {
  margin-top: 100px;
}

label {
  margin: 0;
  padding: 10px;
  cursor: pointer;
}

.toggle ~ .word {
  opacity: 0;
  transition: opacity .3s;
}

.toggle:checked ~ .word {
  opacity: 1;
}

.back-btn {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  left: 0;
  color: #0075ff;
  padding: 20px;
}

.words-container {
  text-align: left;
  margin: 80px 0 0 50px;
}

.top-bar {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  background: white;
  border-bottom: solid 1px #d2d2d2;
  z-index: 2;
}

</style>