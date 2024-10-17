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
const totalQuestions = ref(0)
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

const triviaQuestions = [
  {
    question: "In which book is the verse 'For God so loved the world...' found?",
    options: ['John', 'Genesis', 'Romans', 'Psalms'],
    answer: 'John',
  },
  {
    question: "Complete this verse: 'The Lord is my shepherd, I shall not...'",
    options: ['want', 'fear', 'sin', 'doubt'],
    answer: 'want',
  },
  {
    question: "'I can do all things through Christ who strengthens me' is found in which book?",
    options: ['Philippians', 'James', 'Matthew', 'Isaiah'],
    answer: 'Philippians',
  },
  {
    question: "Which book begins with 'In the beginning God created the heavens and the earth'?",
    options: ['Genesis', 'John', 'Isaiah', 'Romans'],
    answer: 'Genesis',
  },
  {
    question: "Where is the verse 'The fruit of the Spirit is love, joy, peace, patience, kindness...' found?",
    options: ['Galatians', 'Ephesians', 'Romans', 'Philippians'],
    answer: 'Galatians',
  },
]

const startNewGame = () => {
  currentQuestion.value = triviaQuestions[0]
  totalQuestions.value = triviaQuestions.length
  score.value = 0
  gameOver.value = false
  currentQuestionIndex.value = 0
}

const submitAnswer = () => {
  if (selectedAnswer.value === currentQuestion.value.answer) {
    score.value += 1
  }

  answerSubmitted.value = true
}

const nextQuestion = () => {
  currentQuestionIndex.value += 1

  if (currentQuestionIndex.value < triviaQuestions.length) {
    currentQuestion.value = triviaQuestions[currentQuestionIndex.value]
    selectedAnswer.value = null
    answerSubmitted.value = false
  } else {
    gameOver.value = true
  }
}
</script>

<template>
  <div :class="[
    'min-h-screen transition-colors duration-300',
    darkMode ? 'bg-gray-900 text-gray-100' : 'bg-amber-50 text-gray-900'
  ]">
    <div class="container mx-auto px-4 py-8">
      <div class="max-w-2xl mx-auto">
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
          <div :class="[
            'rounded-lg shadow-lg p-6 mb-8',
            darkMode ? 'bg-gray-800' : 'bg-white'
          ]">
            <div class="grid grid-cols-3 gap-4 mb-4">
              <input 
                v-model="book" 
                type="text" 
                placeholder="Book (e.g., john)"
                :class="[
                  'col-span-1 p-2 border rounded focus:ring-2',
                  darkMode ? 
                    'bg-gray-700 border-gray-600 text-white focus:ring-amber-500' : 
                    'bg-amber-50 border-amber-200 focus:ring-amber-500'
                ]"
              >
              <input 
                v-model="chapter" 
                type="number" 
                placeholder="Chapter"
                :class="[
                  'col-span-1 p-2 border rounded focus:ring-2',
                  darkMode ? 
                    'bg-gray-700 border-gray-600 text-white focus:ring-amber-500' : 
                    'bg-amber-50 border-amber-200 focus:ring-amber-500'
                ]"
              >
              <input 
                v-model="verse" 
                type="number" 
                placeholder="Verse"
                :class="[
                  'col-span-1 p-2 border rounded focus:ring-2',
                  darkMode ? 
                    'bg-gray-700 border-gray-600 text-white focus:ring-amber-500' : 
                    'bg-amber-50 border-amber-200 focus:ring-amber-500'
                ]"
              >
            </div>
            
            <div class="flex gap-4">
              <button 
                @click="getVerse"
                :class="[
                  'px-4 py-2 rounded hover:opacity-90 flex-1 transition-colors duration-200 font-serif',
                  darkMode ? 'bg-amber-600 text-white' : 'bg-amber-800 text-white'
                ]"
              >
                Seek Verse
              </button>
              <button 
                @click="getRandomVerse"
                :class="[
                  'px-4 py-2 rounded hover:opacity-90 flex-1 transition-colors duration-200 font-serif',
                  darkMode ? 'bg-amber-700 text-white' : 'bg-amber-700 text-white'
                ]"
              >
                Random Verse
              </button>
            </div>
          </div>

          <div v-if="loading" class="text-center">
            <div class="animate-pulse">Seeking wisdom...</div>
          </div>
          
          <div 
            v-else-if="error" 
            :class="[
              'border-l-4 p-4 rounded',
              darkMode ? 'bg-red-900 border-red-500 text-red-200' : 'bg-red-100 border-red-500 text-red-700'
            ]"
          >
            {{ error }}
          </div>

          <div 
            v-else-if="currentVerse" 
            :class="[
              'rounded-lg shadow-lg p-6 transition-all duration-300',
              darkMode ? 'bg-gray-800' : 'bg-white'
            ]"
          >
            <p class="text-xl mb-4 italic leading-relaxed font-serif">
              "{{ currentVerse.text }}"
            </p>
            <p :class="[
              'text-right font-semibold font-serif',
              darkMode ? 'text-amber-400' : 'text-amber-800'
            ]">
              — {{ currentVerse.reference }}
            </p>
          </div>
        </div>

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

          <!-- Rest of the trivia section with similar theme styling -->
          
        </div>
      </div>
    </div>
  </div>
</template>

<style>

@import url('https://fonts.googleapis.com/css2?family=Crimson+Text:ital,wght@0,400;0,600;1,400&display=swap');

.font-serif {
  font-family: 'Crimson Text', serif;
}


:root {
  color-scheme: light dark;
}


input[type="number"]::-webkit-inner-spin-button,
input[type="number"]::-webkit-outer-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

input[type="number"] {
  -moz-appearance: textfield;
}
</style>