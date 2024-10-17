<script setup>
import { ref, computed } from 'vue'

const darkMode = ref(false)
const book = ref('')
const chapter = ref('')
const verse = ref('')
const currentVerse = ref(null)
const loading = ref(false)
const error = ref(false)
const showTrivia = ref(false)
const currentQuestion = ref(null)
const options = ref([])
const score = ref(0)
const currentQuestionIndex = ref(0)
const triviaLoading = ref(false)
const gameOver = ref(false)
const selectedAnswer = ref(null)
const answerSubmitted = ref(false)

const randomBooks = [
  'john', 'genesis', 'psalms', 'proverbs', 'matthew',
  'romans', 'philippians', 'james', 'isaiah', 'exodus'
]

const toggleTheme = () => {
  darkMode.value = !darkMode.value
  document.documentElement.classList.toggle('dark')
}

// Verse fetching functions
const getVerse = async () => {
  if (!book.value || !chapter.value || !verse.value) {
    error.value = 'Please fill in all fields'
    return
  }

  loading.value = true
  error.value = null

  try {
    const response = await fetch(
      `https://bible-api.com/${book.value}+${chapter.value}:${verse.value}`
    )
   
    if (!response.ok) {
      throw new Error('Verse not found')
    }

    const data = await response.json()
    currentVerse.value = {
      text: data.text,
      reference: `${data.reference}`
    }
  } catch (err) {
    error.value = 'Error fetching verse. Please check your input and try again.'
  } finally {
    loading.value = false
  }
}

const getRandomVerse = async () => {
  book.value = randomBooks[Math.floor(Math.random() * randomBooks.length)]
  chapter.value = Math.floor(Math.random() * 20) + 1
  verse.value = Math.floor(Math.random() * 30) + 1
  await getVerse()
}

// Bible Trivia functions
const bibleQuestions = [
  {
    question: "Who was the first man created by God?",
    options: ["Adam", "Noah", "Abraham", "Moses"],
    correctAnswer: "Adam"
  },
  {
    question: "Which prophet was swallowed by a great fish?",
    options: ["Elijah", "Jonah", "Isaiah", "Jeremiah"],
    correctAnswer: "Jonah"
  },
  {
    question: "Who denied Jesus three times before the rooster crowed?",
    options: ["John", "Peter", "James", "Andrew"],
    correctAnswer: "Peter"
  },
  {
    question: "What was the name of Abraham's wife?",
    options: ["Rachel", "Rebekah", "Sarah", "Leah"],
    correctAnswer: "Sarah"
  },
  {
    question: "Which book of the Bible tells the story of the Exodus?",
    options: ["Genesis", "Exodus", "Leviticus", "Numbers"],
    correctAnswer: "Exodus"
  }
]

const startNewGame = () => {
  score.value = 0
  currentQuestionIndex.value = 0
  gameOver.value = false
  loadQuestion()
}

const loadQuestion = () => {
  triviaLoading.value = true
  currentQuestion.value = bibleQuestions[currentQuestionIndex.value]
  options.value = currentQuestion.value.options
  selectedAnswer.value = null
  answerSubmitted.value = false
  triviaLoading.value = false
}

const submitAnswer = () => {
  if (!selectedAnswer.value) return
  answerSubmitted.value = true
  if (selectedAnswer.value === currentQuestion.value.correctAnswer) {
    score.value++
  }
}

const nextQuestion = () => {
  if (currentQuestionIndex.value < bibleQuestions.length - 1) {
    currentQuestionIndex.value++
    loadQuestion()
  } else {
    gameOver.value = true
  }
}

const totalQuestions = computed(() => bibleQuestions.length)

</script>

<template>
  <div :class="[
    'min-h-screen transition-colors duration-300',
    darkMode ? 'bg-gray-900 text-gray-100' : 'bg-amber-50 text-gray-900'
  ]">
    <div class="container mx-auto px-4 py-8">
      <div class="max-w-2xl mx-auto">
        <!-- Theme Toggle -->
        <div class="flex justify-end mb-4">
          <button
            @click="toggleTheme"
            class="p-2 rounded-full transition-colors duration-200"
            :class="darkMode ? 'bg-gray-700 text-yellow-400' : 'bg-amber-100 text-gray-900'"
          >
            <span v-if="darkMode">☀️</span>
            <span v-else>🌙</span>
          </button>
        </div>

        <!-- Header -->
        <div class="text-center mb-8">
          <h1 :class="[
            'text-4xl font-serif font-bold mb-2',
            darkMode ? 'text-amber-400' : 'text-amber-800'
          ]">
            Sacred Scripture
          </h1>
          <p :class="[
            'text-lg italic',
            darkMode ? 'text-gray-400' : 'text-amber-700'
          ]">
            "Thy word is a lamp unto my feet, and a light unto my path"
          </p>
        </div>

        <!-- Navigation Tabs -->
        <div class="flex justify-center mb-8 gap-4">
          <button
            @click="showTrivia = false"
            :class="[
              'px-6 py-3 rounded-lg font-serif transition-colors duration-200',
              !showTrivia ?
                (darkMode ? 'bg-amber-600 text-white' : 'bg-amber-800 text-white') :
                (darkMode ? 'bg-gray-700 text-gray-300' : 'bg-amber-100 text-amber-900')
            ]"
          >
            Scripture Search
          </button>
          <button
            @click="showTrivia = true"
            :class="[
              'px-6 py-3 rounded-lg font-serif transition-colors duration-200',
              showTrivia ?
                (darkMode ? 'bg-amber-600 text-white' : 'bg-amber-800 text-white') :
                (darkMode ? 'bg-gray-700 text-gray-300' : 'bg-amber-100 text-amber-900')
            ]"
          >
            Scripture Trivia
          </button>
        </div>

        <div v-if="!showTrivia">
          <!-- Verse Search Section -->
          <!-- ... [Keep the existing verse search section unchanged] ... -->
        </div>

        <!-- Trivia Section -->
        <div v-else :class="[
          'rounded-lg shadow-lg p-6',
          darkMode ? 'bg-gray-800' : 'bg-white'
        ]">
          <div v-if="!currentQuestion && !gameOver" class="text-center">
            <h2 class="text-2xl font-bold font-serif mb-4">Scripture Knowledge</h2>
            <p class="mb-4">Test your understanding of Holy Scripture</p>
            <button
              @click="startNewGame"
              :class="[
                'px-6 py-3 rounded hover:opacity-90 transition-colors duration-200 font-serif',
                darkMode ? 'bg-amber-600 text-white' : 'bg-amber-800 text-white'
              ]"
            >
              Begin Journey
            </button>
          </div>

          <div v-else-if="triviaLoading" class="text-center">
            <p class="animate-pulse">Preparing question...</p>
          </div>

          <div v-else-if="gameOver" class="text-center">
            <h2 class="text-2xl font-bold font-serif mb-4">Journey Complete</h2>
            <p class="text-xl mb-4">Your score: {{ score }} / {{ totalQuestions }}</p>
            <button
              @click="startNewGame"
              :class="[
                'px-6 py-3 rounded hover:opacity-90 transition-colors duration-200 font-serif',
                darkMode ? 'bg-amber-600 text-white' : 'bg-amber-800 text-white'
              ]"
            >
              Begin New Journey
            </button>
          </div>

          <div v-else>
            <h3 class="font-serif text-xl mb-6">{{ currentQuestion.question }}</h3>
            <div class="mb-6 space-y-4">
              <div v-for="option in options" :key="option" class="flex items-center">
                <input
                  type="radio"
                  :id="option"
                  :value="option"
                  v-model="selectedAnswer"
                  :disabled="answerSubmitted"
                  class="mr-3"
                />
                <label :for="option" class="font-serif text-lg cursor-pointer">{{ option }}</label>
              </div>
            </div>

            <div class="flex justify-between items-center">
              <button
                @click="submitAnswer"
                :disabled="!selectedAnswer || answerSubmitted"
                :class="[
                  'px-6 py-3 rounded transition-colors duration-200 font-serif',
                  (!selectedAnswer || answerSubmitted) ?
                    (darkMode ? 'bg-gray-600 text-gray-400' : 'bg-gray-300 text-gray-600') :
                    (darkMode ? 'bg-green-600 text-white hover:bg-green-700' : 'bg-green-700 text-white hover:bg-green-800')
                ]"
              >
                Submit Answer
              </button>

              <p class="font-serif">
                Question {{ currentQuestionIndex + 1 }} of {{ totalQuestions }}
              </p>
            </div>

            <div v-if="answerSubmitted" class="mt-6">
              <p v-if="selectedAnswer === currentQuestion.correctAnswer" class="text-green-600 font-serif text-lg">
                Correct! Well done.
              </p>
              <p v-else class="text-red-600 font-serif text-lg">
                Incorrect. The correct answer is: {{ currentQuestion.correctAnswer }}
              </p>
              <button 
                @click="nextQuestion" 
                :class="[
                  'mt-4 px-6 py-3 rounded transition-colors duration-200 font-serif',
                  darkMode ? 'bg-amber-600 text-white hover:bg-amber-700' : 'bg-amber-800 text-white hover:bg-amber-900'
                ]"
              >
                Next Question
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
/* ... [Keep the existing styles unchanged] ... */
</style>