<script setup>
import { ref, computed } from 'vue'
import data from '@/assets/data/nouns-declension-cz.json'

// ✅ Case labels (corrected)
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

// ✅ State
const question = ref(null)
const selectedAnswers = ref(new Set())
const answerIsCorrect = ref(false)
const askedQuestions = ref(new Set())

const totalAttempts = ref(0)
const totalCorrect = ref(0)

const totalWrong = computed(() => totalAttempts.value - totalCorrect.value)
const successRate = computed(() => {
  if (totalAttempts.value === 0) return 0
  return Math.round((totalCorrect.value / totalAttempts.value) * 100)
})

// ✅ Option generation
function generateOption(solution, pool) {
  let options = new Set()
  options.add(solution)
  while (options.size < 3) {
    const randomIndex = Math.floor(Math.random() * pool.length)
    options.add(pool[randomIndex])
  }
  return Array.from(options).sort(() => Math.random() - 0.5)
}

// ✅ Question generation
function getNewQuestion() {
  const maxTries = 1000
  let tries = 0

  while (tries < maxTries) {
    tries++
    const randomNoun = data[Math.floor(Math.random() * data.length)]
    const baseForm = randomNoun.NOMsg
    const declensions = { ...randomNoun }
    delete declensions.NOMsg

    const keys = Object.keys(declensions)
    const caseToAsk = keys[Math.floor(Math.random() * keys.length)]

    const key = `${baseForm}|${caseToAsk}`
    if (!askedQuestions.value.has(key)) {
      askedQuestions.value.add(key)
      const solution = declensions[caseToAsk]
      const options = generateOption(solution, Object.values(declensions))

      return { baseForm, caseToAsk, solution, options }
    }
  }

  return null // All exhausted
}

// ✅ Quiz control
function resetQuiz() {
  const next = getNewQuestion()
  if (next) {
    question.value = next
    selectedAnswers.value.clear()
    answerIsCorrect.value = false
  } else {
    question.value = null
  }
}

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

// ✅ Bonus: Reset full quiz
function resetAll() {
  askedQuestions.value.clear()
  totalAttempts.value = 0
  totalCorrect.value = 0
  selectedAnswers.value.clear()
  answerIsCorrect.value = false
  question.value = getNewQuestion()
}

// ✅ Initialize first question
resetQuiz()
</script>

<template>
  <div class="content-container">
    <main>
      <h1>Grammatiktrainer Tschechisch – Deklination der Nomen</h1>

      <div class="quiz" v-if="question">
        <p>
          <strong>{{ question.baseForm }}</strong> im
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

      <div class="quiz" v-else>
        <p>✅ Du hast alle möglichen Fragen durchgespielt!</p>
      </div>
    </main>

    <aside>
      <h2>Statistik</h2>
      <p><strong>{{ totalAttempts }}</strong> Aufgaben</p>
      <p><strong>{{ totalCorrect }}</strong> richtig</p>
      <p><strong>{{ totalWrong }}</strong> falsch</p>
      <p><strong>{{ successRate }}%</strong> korrekt</p>
      <button class="reset-button" @click="resetAll">
        Quiz zurücksetzen
      </button>
    </aside>
  </div>
</template>

<style scoped lang="scss">
.content-container {
  display: flex;
  background-color: #fff;
  min-height: 90vh;
  flex-direction: column;
  padding: 1rem;
  gap: 2rem;

  @media (min-width: 768px) {
    flex-direction: row;
    align-items: flex-start;
    padding: 2rem;
  }
}

main {
  flex: 2;
  justify-content: center;
  text-align: center;
  h1 {
    text-align: center;
    font-size: 1.125rem;
    margin-bottom: 1.5rem;

    @media (min-width: 768px) {
      font-size: 2rem;
    }
  }

  .quiz {
    display: flex;
    flex-direction: column;
    align-items: center;
    font-size: 1rem;
    @media (min-width: 768px) {
      font-size: 1.25rem;
    }
  }

  .options {
    display: flex;
    flex-wrap: wrap;
    gap: 0.75rem;
    margin-top: 1.25rem;

    button {
      padding: 0.5rem 1rem;
      font-size: 0.9rem;
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

      @media (min-width: 768px) {
        padding: 0.75rem 1.25rem;
        font-size: 1rem;
      }
    }
  }

  .feedback {
    margin-top: 1.25rem;
    font-weight: bold;

    &.correct {
      color: #4caf50;
    }
  }

  .next-button {
    margin-top: 1.25rem;
    padding: 0.6rem 1.2rem;
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
  border-top: 1px solid #ddd;
  padding-top: 1rem;
  font-size: 0.95rem;
  color: #444;

  @media (min-width: 768px) {
    border-top: none;
    border-left: 1px solid #ddd;
    padding-top: 0;
    padding-left: 2rem;
  }


  h2 {
    font-size: 1.1rem;
    margin-bottom: 1rem;

    @media (min-width: 768px) {
      font-size: 1.25rem;
    }
  }

  p {
    margin: 0.5rem 0;
  }

  .reset-button {
    margin-top: 1rem;
    padding: 0.5rem 1rem;
    font-size: 0.9rem;
    background-color: #9e9e9e;
    color: white;
    border: none;
    border-radius: 0.5rem;
    cursor: pointer;

    &:hover {
      background-color: #757575;
    }

    @media (min-width: 768px) {
      font-size: 1rem;
    }
  }
}
</style>

