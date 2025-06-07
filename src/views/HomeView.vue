<script setup>
import { ref, computed } from 'vue'
import TheWelcome from '../components/TheWelcome.vue'
import data from '@/assets/data/nouns-declension-cz.json'

function selectRandomNoun() {
  const nouns = data;
  const randomIndex = Math.floor(Math.random() * nouns.length);
  return nouns[randomIndex];
}
const nounInTest = ref(selectRandomNoun());

function generateOption(solution, pool) {
  let options = new Set();
  options.add(solution);
  while (options.size < 3) {
    const randomIndex = Math.floor(Math.random() * pool.length);
    options.add(pool[randomIndex]);
  }
  let optionsArray = Array.from(options);
  optionsArray.sort(() => Math.random() - 0.5); // Shuffle the options
  return optionsArray;
  
}

const question = computed(() => {
  const testNoun = JSON.parse(JSON.stringify(nounInTest.value)); // Deep copy to avoid mutation
  const baseForm = nounInTest.value.NOMsg;
  delete testNoun.NOMsg; // Remove the base form from the test object
  const randomIndex = Math.floor(Math.random() * Object.keys(testNoun).length);
  const caseToAsk = Object.keys(testNoun)[randomIndex];
  const solution = testNoun[caseToAsk];
  const options = generateOption(solution, Object.values(testNoun));
  return {baseForm, caseToAsk, solution, options};
});

</script>

<template>
  <main>
    <h1>Grammatiktrainer Tschechisch - Deklination der Nomen</h1>
    <pre v-if="false">{{ nounInTest }}</pre>
    <pre v-if="true">{{ question }}</pre>
    <pre v-if="false">{{ data }}</pre>
  </main>
</template>

<style scoped lang="scss">
main {
  h1 {
    text-align: center;
    margin-top: 2rem;
    font-size: 2.5rem;
    color: #333;
  }

}

</style>
