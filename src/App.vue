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
  <div class="min-h-screen bg-gray-100">
    <div class="container mx-auto px-4 py-8">
      <div class="max-w-2xl mx-auto">
        <h1 class="text-3xl font-bold text-center mb-8 text-indigo-600">
          Bible Study App
        </h1>

        <div class="flex justify-center mb-8 gap-4">
          <button 
            @click="showTrivia = false" 
            :class="[
              'px-4 py-2 rounded transition-colors duration-200',
              !showTrivia ? 'bg-indigo-600 text-white' : 'bg-gray-200 text-gray-700'
            ]"
          >
            Verse Generator
          </button>
          <button 
            @click="showTrivia = true" 
            :class="[
              'px-4 py-2 rounded transition-colors duration-200',
              showTrivia ? 'bg-indigo-600 text-white' : 'bg-gray-200 text-gray-700'
            ]"
          >
            Trivia Game
          </button>
        </div>

        <div v-if="!showTrivia">
          <div class="bg-white rounded-lg shadow-md p-6 mb-8">
            <div class="grid grid-cols-3 gap-4 mb-4">
              <input 
                v-model="book" 
                type="text" 
                placeholder="Book (e.g., john)"
                class="col-span-1 p-2 border rounded focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500"
              >
              <input 
                v-model="chapter" 
                type="number" 
                placeholder="Chapter"
                class="col-span-1 p-2 border rounded focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500"
              >
              <input 
                v-model="verse" 
                type="number" 
                placeholder="Verse"
                class="col-span-1 p-2 border rounded focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500"
              >
            </div>
            
            <div class="flex gap-4">
              <button 
                @click="getVerse"
                class="bg-indigo-600 text-white px-4 py-2 rounded hover:bg-indigo-700 flex-1 transition-colors duration-200"
              >
                Get Verse
              </button>
              <button 
                @click="getRandomVerse"
                class="bg-green-600 text-white px-4 py-2 rounded hover:bg-green-700 flex-1 transition-colors duration-200"
              >
                Random Verse
              </button>
            </div>
          </div>

          <div v-if="loading" class="text-center text-gray-600">
            <div class="animate-pulse">Loading...</div>
          </div>
          
          <div 
            v-else-if="error" 
            class="bg-red-100 border-l-4 border-red-500 text-red-700 p-4 rounded"
          >
            {{ error }}
          </div>

          <div 
            v-else-if="currentVerse" 
            class="bg-white rounded-lg shadow-md p-6 transition-all duration-300"
          >
            <p class="text-xl mb-4 italic leading-relaxed">
              {{ currentVerse.text }}
            </p>
            <p class="text-right text-indigo-600 font-semibold">
              {{ currentVerse.reference }}
            </p>
          </div>
        </div>

        <div v-else class="bg-white rounded-lg shadow-md p-6">
          <div v-if="!currentQuestion && !gameOver" class="text-center">
            <h2 class="text-xl font-bold mb-4">Bible Verse Trivia</h2>
            <p class="mb-4">Test your knowledge of Bible verses! Match the verse to its correct reference.</p>
            <button 
              @click="startNewGame"
              class="bg-indigo-600 text-white px-6 py-3 rounded hover:bg-indigo-700 transition-colors duration-200"
            >
              Start Game
            </button>
          </div>

          <div v-else-if="gameOver" class="text-center">
            <h2 class="text-2xl font-bold mb-4">Game Over!</h2>
            <p class="text-xl mb-4">Your Score: {{ score }} / {{ totalQuestions }}</p>
            <p class="text-lg mb-6">Percentage: {{ scorePercentage }}%</p>
            <button 
              @click="startNewGame"
              class="bg-indigo-600 text-white px-6 py-3 rounded hover:bg-indigo-700 transition-colors duration-200"
            >
              Play Again
            </button>
          </div>

          <div v-else>
            <div class="mb-4">
              <p class="text-sm text-gray-600 mb-2">Question {{ totalQuestions + 1 }} of 5</p>
              <p class="text-lg mb-2">Match this verse to its reference:</p>
              <p class="italic text-xl mb-6">{{ currentQuestion.text }}</p>
            </div>

            <div class="space-y-3">
              <button
                v-for="option in options"
                :key="option"
                @click="checkAnswer(option)"
                class="w-full text-left p-3 rounded border hover:bg-indigo-50 transition-colors duration-200"
              >
                {{ option }}
              </button>
            </div>

            <div class="mt-4 text-right text-gray-600">
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
</style>