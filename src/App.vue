<script setup>
import { ref, computed } from 'vue'

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

const randomBooks = [
  'john', 'genesis', 'psalms', 'proverbs', 'matthew',
  'romans', 'philippians', 'james', 'isaiah', 'exodus'
]

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

const generateQuestion = async () => {
  triviaLoading.value = true
  
  try {
    const randomBook = randomBooks[Math.floor(Math.random() * randomBooks.length)]
    const randomChapter = Math.floor(Math.random() * 20) + 1
    const randomVerse = Math.floor(Math.random() * 30) + 1
    
    const response = await fetch(
      `https://bible-api.com/${randomBook}+${randomChapter}:${randomVerse}`
    )
    
    if (!response.ok) {
      throw new Error('Failed to fetch verse')
    }

    const data = await response.json()
    
    const wrongOptions = [
      `${randomBooks[Math.floor(Math.random() * randomBooks.length)]} ${Math.floor(Math.random() * 20) + 1}:${Math.floor(Math.random() * 30) + 1}`,
      `${randomBooks[Math.floor(Math.random() * randomBooks.length)]} ${Math.floor(Math.random() * 20) + 1}:${Math.floor(Math.random() * 30) + 1}`,
      `${randomBooks[Math.floor(Math.random() * randomBooks.length)]} ${Math.floor(Math.random() * 20) + 1}:${Math.floor(Math.random() * 30) + 1}`
    ]

    const allOptions = [...wrongOptions, data.reference]
    const shuffledOptions = allOptions.sort(() => Math.random() - 0.5)

    currentQuestion.value = {
      text: data.text,
      correctAnswer: data.reference
    }
    options.value = shuffledOptions
    
  } catch (err) {
    error.value = 'Error generating trivia question. Please try again.'
  } finally {
    triviaLoading.value = false
  }
}

const checkAnswer = (selectedAnswer) => {
  totalQuestions.value++
  
  if (selectedAnswer === currentQuestion.value.correctAnswer) {
    score.value++
  }
  
  if (totalQuestions.value >= 5) {
    gameOver.value = true
  } else {
    generateQuestion()
  }
}

const startNewGame = () => {
  score.value = 0
  totalQuestions.value = 0
  gameOver.value = false
  generateQuestion()
}

const scorePercentage = computed(() => {
  return totalQuestions.value > 0 
    ? Math.round((score.value / totalQuestions.value) * 100) 
    : 0
})
</script>

<template>
  <div class="min-h-screen bg-gradient-to-b from-purple-500 to-indigo-800 text-white flex items-center justify-center">
    <div class="container mx-auto px-4 py-8">
      <div class="max-w-2xl mx-auto backdrop-blur-md bg-white/30 rounded-lg shadow-2xl p-8 border border-gray-200/30">
        <h1 class="text-4xl font-extrabold text-center mb-8 text-white tracking-wider drop-shadow-lg">
          Bible Study App
        </h1>

        <!-- Tab Toggle -->
        <div class="flex justify-center mb-8 gap-4">
          <button 
            @click="showTrivia = false" 
            :class="[ 
              'px-6 py-3 rounded-lg font-semibold transform transition-transform duration-300 hover:scale-105 shadow-md',
              !showTrivia ? 'bg-purple-700 text-white shadow-lg hover:shadow-xl' : 'bg-white/50 text-gray-800 hover:shadow-md' 
            ]"
          >
            Verse Generator
          </button>
          <button 
            @click="showTrivia = true" 
            :class="[ 
              'px-6 py-3 rounded-lg font-semibold transform transition-transform duration-300 hover:scale-105 shadow-md',
              showTrivia ? 'bg-purple-700 text-white shadow-lg hover:shadow-xl' : 'bg-white/50 text-gray-800 hover:shadow-md' 
            ]"
          >
            Trivia Game
          </button>
        </div>

        <!-- Verse Generator -->
        <div v-if="!showTrivia">
          <div class="backdrop-blur-md bg-white/40 rounded-lg shadow-md p-6 mb-8 border border-gray-300/20">
            <div class="grid grid-cols-3 gap-4 mb-4">
              <input 
                v-model="book" 
                type="text" 
                placeholder="Book (e.g., john)"
                class="col-span-1 p-3 border rounded-lg focus:ring-4 focus:ring-purple-500 focus:border-purple-500 shadow-sm backdrop-blur-sm bg-white/70 text-gray-900"
              >
              <input 
                v-model="chapter" 
                type="number" 
                placeholder="Chapter"
                class="col-span-1 p-3 border rounded-lg focus:ring-4 focus:ring-purple-500 focus:border-purple-500 shadow-sm backdrop-blur-sm bg-white/70 text-gray-900"
              >
              <input 
                v-model="verse" 
                type="number" 
                placeholder="Verse"
                class="col-span-1 p-3 border rounded-lg focus:ring-4 focus:ring-purple-500 focus:border-purple-500 shadow-sm backdrop-blur-sm bg-white/70 text-gray-900"
              >
            </div>
            
            <div class="flex gap-4">
              <button 
                @click="getVerse"
                class="bg-purple-700 text-white px-4 py-3 rounded-lg hover:bg-purple-800 transition-colors duration-300 shadow-lg hover:shadow-xl flex-1 transform hover:scale-105"
              >
                Get Verse
              </button>
              <button 
                @click="getRandomVerse"
                class="bg-green-600 text-white px-4 py-3 rounded-lg hover:bg-green-700 transition-colors duration-300 shadow-lg hover:shadow-xl flex-1 transform hover:scale-105"
              >
                Random Verse
              </button>
            </div>
          </div>

          <!-- Loading -->
          <div v-if="loading" class="text-center text-gray-600">
            <div class="animate-spin h-10 w-10 border-t-4 border-purple-600 rounded-full mx-auto"></div>
          </div>

          <!-- Error -->
          <div 
            v-else-if="error" 
            class="bg-red-100 border-l-4 border-red-500 text-red-700 p-4 rounded"
          >
            {{ error }}
          </div>

          <!-- Verse -->
          <div 
            v-else-if="currentVerse" 
            class="backdrop-blur-lg bg-white/50 rounded-lg shadow-md p-6 transition-all duration-300 border border-gray-300/20"
          >
            <p class="text-xl mb-4 italic leading-relaxed text-gray-900">
              "{{ currentVerse.text }}"
            </p>
            <p class="text-right text-purple-700 font-semibold">
              {{ currentVerse.reference }}
            </p>
          </div>
        </div>

        <!-- Trivia Game -->
        <div v-else class="backdrop-blur-md bg-white/40 rounded-lg shadow-lg p-6 border border-gray-200/30">
          <!-- Start Screen -->
          <div v-if="!currentQuestion && !gameOver" class="text-center">
            <h2 class="text-xl font-bold mb-4 text-white">Bible Verse Trivia</h2>
            <p class="mb-4 text-gray-100">Test your knowledge of Bible verses! Match the verse to its correct reference.</p>
            <button 
              @click="startNewGame"
              class="bg-purple-700 text-white px-6 py-3 rounded-lg hover:bg-purple-800 transition-colors duration-300 shadow-lg hover:shadow-xl transform hover:scale-105"
            >
              Start Game
            </button>
          </div>

          <!-- Game Over -->
          <div v-else-if="gameOver" class="text-center">
            <h2 class="text-2xl font-bold mb-4 text-white">Game Over!</h2>
            <p class="text-xl mb-4 text-gray-100">Your Score: {{ score }} / {{ totalQuestions }}</p>
            <p class="text-lg mb-6 text-gray-100">Percentage: {{ scorePercentage }}%</p>
            <button 
              @click="startNewGame"
              class="bg-purple-700 text-white px-6 py-3 rounded-lg hover:bg-purple-800 transition-colors duration-300 shadow-lg hover:shadow-xl transform hover:scale-105"
            >
              Play Again
            </button>
          </div>

          <!-- Trivia Question -->
          <div v-else>
            <div class="mb-4">
              <p class="text-sm text-gray-200 mb-2">Question {{ totalQuestions + 1 }} of 5</p>
              <p class="text-lg mb-2 text-white">Match this verse to its reference:</p>
              <p class="italic text-xl mb-6 text-gray-100">{{ currentQuestion.text }}</p>
            </div>

            <div class="space-y-3">
              <button
                v-for="option in options"
                :key="option"
                @click="checkAnswer(option)"
                class="w-full text-left p-3 rounded-lg border hover:bg-purple-100 transition-colors duration-300 shadow-sm"
              >
                {{ option }}
              </button>
            </div>

            <div class="mt-4 text-right text-gray-300">
              Score: {{ score }} / {{ totalQuestions }}
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
input[type="number"]::-webkit-inner-spin-button,
input[type="number"]::-webkit-outer-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

input[type="number"] {
  -moz-appearance: textfield;
}

html, body {
  min-height: 100%;
}
</style>
