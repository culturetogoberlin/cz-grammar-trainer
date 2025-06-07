<script setup>
import { ref, computed } from 'vue'
import data from '@/assets/data/nouns-declension-cz.json'

// 1. Correct case labels
const caseLabels = {
  NOMsg: 'Nominativ Singular',
  NOMpl: 'Nominativ Plural',
  GENsg: 'Genitiv Singular',
  GENpl: 'Genitiv Plural',
  DATsg: 'Dativ Singular',
  DATpl: 'Dativ Plural',
  AKKsg: 'Akkusativ Singular',
  AKKpl: 'Akkusativ Plural',
  LOKsg: 'Lokativ Singular',
  LOKpl: 'Lokativ Plural',
  INSsg: 'Instrumental Singular',
  INSpl: 'Instrumental Plural',
  VOKsg: 'Vokativ Singular',
  VOKpl: 'Vokativ Plural',
}

const nounInTest = ref(getRandomNoun())
const selectedAnswers = ref(new Set())
const answerIsCorrect = ref(false)

const totalAttempts = ref(0)
const totalCorrect = ref(0)
const totalWrong = computed(() => totalAttempts.value - totalCorrect.value)
const successRate = computed(() => {
  if (totalAttempts.value === 0) return 0
  return Math.round((totalCorrect.value / totalAttempts.value) * 100)
})

function getRandomNoun() {
  const randomIndex = Math.floor(Math.random() * data.length)
  return data[randomIndex]
}

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
  if (answerIsCorrect.value || selectedAnswers.value.has(option)) return

  selectedAnswers.value.add(option)

  if (option === question.value.solution) {
    answerIsCorrect.value = true
    totalCorrect.value++
    totalAttempts.value++
  } else {
    if (selectedAnswers.value.size === 1) {
      totalAttempts.value++
    }
  }
}

function resetQuiz() {
  nounInTest.value = getRandomNoun()
  selectedAnswers.value.clear()
  answerIsCorrect.value = false
}
</script>

<template>
  <div class="content-container">
    <main>
      <h1>Grammatiktrainer Tschechisch – Deklination der Nomen</h1>

      <div class="quiz">
        <p>
          Wie lautet die Form von <strong>{{ question.baseForm }}</strong> im
          <strong>{{ caseLabels[question.caseToAsk] }}</strong>?
        </p>

        <div class="options">
          <button
            v-for="(opt, index) in question.options"
            :key="index"
            @click="selectOption(opt)"
            :class="{
              wrong: selectedAnswers.has(opt) && opt !== question.solution,
              correct: answerIsCorrect && opt === question.solution
            }"
            :disabled="answerIsCorrect || selectedAnswers.has(opt)"
          >
            {{ opt }}
          </button>
        </div>

        <p v-if="answerIsCorrect" class="feedback correct">
          ✔️ Richtig! Gut gemacht.
        </p>

        <button v-if="answerIsCorrect" class="next-button" @click="resetQuiz">
          Nächstes Substantiv
        </button>
      </div>
    </main>

    <aside>
      <h2>Statistik</h2>
      <p><strong>{{ totalAttempts }}</strong> Aufgaben</p>
      <p><strong>{{ totalCorrect }}</strong> richtig</p>
      <p><strong>{{ totalWrong }}</strong> falsch</p>
      <p><strong>{{ successRate }}%</strong> korrekt</p>
    </aside>
  </div>
</template>

<style scoped lang="scss">
.content-container {
  display: flex;
  gap: 2rem;
  flex-direction: row;
  align-items: flex-start;
  padding: 2rem;
}

main {
  flex: 2;
  h1 {
    text-align: center;
    font-size: 2rem;
    margin-bottom: 2rem;
  }

  .quiz {
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
  }

  .next-button {
    margin-top: 1.5rem;
    padding: 0.75rem 1.5rem;
    font-size: 1rem;
    background-color: #1976d2;
    color: white;
    border: none;
    border-radius: 0.5rem;
    cursor: pointer;

    &:hover {
      background-color: #1565c0;
    }
  }
}

aside {
  flex: 1;
  border-left: 1px solid #ddd;
  padding-left: 2rem;
  font-size: 1rem;
  color: #444;

  h2 {
    font-size: 1.25rem;
    margin-bottom: 1rem;
  }

  p {
    margin: 0.5rem 0;
  }
}
</style>
