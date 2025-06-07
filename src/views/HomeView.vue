<script setup>
import { ref, computed } from 'vue'
import data from '@/assets/data/nouns-declension-cz.json'

function selectRandomNoun() {
  const randomIndex = Math.floor(Math.random() * data.length)
  return data[randomIndex]
}

const nounInTest = ref(selectRandomNoun())
const selectedAnswer = ref(null)
const answerIsCorrect = ref(null)

function generateOption(solution, pool) {
  let options = new Set()
  options.add(solution)
  while (options.size < 3) {
    const randomIndex = Math.floor(Math.random() * pool.length)
    options.add(pool[randomIndex])
  }
  return Array.from(options).sort(() => Math.random() - 0.5)
}

const question = computed(() => {
  const testNoun = JSON.parse(JSON.stringify(nounInTest.value))
  const baseForm = nounInTest.value.NOMsg
  delete testNoun.NOMsg
  const keys = Object.keys(testNoun)
  const randomIndex = Math.floor(Math.random() * keys.length)
  const caseToAsk = keys[randomIndex]
  const solution = testNoun[caseToAsk]
  const options = generateOption(solution, Object.values(testNoun))
  return { baseForm, caseToAsk, solution, options }
})

function selectOption(option) {
  selectedAnswer.value = option
  answerIsCorrect.value = option === question.value.solution
}
</script>

<template>
  <main>
    <h1>Grammatiktrainer Tschechisch – Deklination der Nomen</h1>

    <div class="quiz">
      <p>
        Wie lautet die Form von <strong>{{ question.baseForm }}</strong> im
        <strong>{{ question.caseToAsk }}</strong>?
      </p>

      <div class="options">
        <button
          v-for="(opt, index) in question.options"
          :key="index"
          @click="selectOption(opt)"
          :class="{
            correct: selectedAnswer && opt === question.solution,
            wrong: selectedAnswer && opt === selectedAnswer && opt !== question.solution,
          }"
          :disabled="selectedAnswer"
        >
          {{ opt }}
        </button>
      </div>

      <p v-if="selectedAnswer && answerIsCorrect" class="feedback correct">
        ✔️ Richtig!
      </p>
      <p v-else-if="selectedAnswer && !answerIsCorrect" class="feedback wrong">
        ❌ Leider falsch.
      </p>
    </div>
  </main>
</template>

<style scoped lang="scss">
main {
  padding: 2rem;

  h1 {
    text-align: center;
    font-size: 2rem;
    margin-bottom: 2rem;
  }

  .quiz {
    max-width: 600px;
    margin: 0 auto;
    font-size: 1.25rem;
  }

  .options {
    display: flex;
    flex-wrap: wrap;
    gap: 1rem;
    margin-top: 1.5rem;

    button {
      padding: 0.75rem 1.25rem;
      font-size: 1rem;
      border: none;
      border-radius: 0.5rem;
      background-color: #eee;
      cursor: pointer;
      transition: background-color 0.3s ease;

      &:hover {
        background-color: #ddd;
      }

      &.correct {
        background-color: #4caf50;
        color: white;
      }

      &.wrong {
        background-color: #f44336;
        color: white;
      }

      &:disabled {
        cursor: default;
        opacity: 0.9;
      }
    }
  }

  .feedback {
    margin-top: 1.5rem;
    font-weight: bold;

    &.correct {
      color: #4caf50;
    }

    &.wrong {
      color: #f44336;
    }
  }
}
</style>
