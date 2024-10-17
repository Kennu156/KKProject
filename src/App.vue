<script setup>
import { ref } from 'vue'

const book = ref('')
const chapter = ref('')
const verse = ref('')
const currentVerse = ref(null)
const loading = ref(false)
const error = ref(null)

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
</script>

<template>
  <div class="min-h-screen bg-gray-100">
    <div class="container mx-auto px-4 py-8">
      <div class="max-w-2xl mx-auto">
        <h1 class="text-3xl font-bold text-center mb-8 text-indigo-600">
          Bible Verse Generator
        </h1>

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
