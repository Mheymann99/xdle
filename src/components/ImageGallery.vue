<script setup lang="ts">
import { onBeforeMount, ref, computed } from 'vue'
import { useCounterStore } from '@/stores/counter'

interface Affinity {
  name: string
  symbol: string
  atomicNumber: number
}

interface Character {
  id: string
  name: string
  avatar: string
  affinity: Affinity
}

interface HighlightObject {
  before: string
  match: string
  after: string
}

interface FilteredCharacters {
  character: Character
  highlight: HighlightObject | false
}

const characters = ref<Character[]>([])
const inputText = ref('')
const isFetching = ref(false)
const counter = useCounterStore()
const cheatMode = ref(false)
const correctAnswer = '7'
const win = ref(false)
const incorrectAnswers = ref(new Set())
//const hintCounter = ref(0)

onBeforeMount(async () => {
  isFetching.value = true
  const charactersJson = await fetch('https://6878c5e963f24f1fdc9f63cf.mockapi.io/api/characters')
  characters.value = await charactersJson.json()
  isFetching.value = false
})

const filteredCharacters = computed(() => {
  if (inputText.value.length > 0) {
    return characters.value
      .filter((char) => char.name.toLowerCase().includes(inputText.value.toLowerCase()))
      .sort((a, b) => {
        const substring = inputText.value.toLowerCase()
        const indexA = a.name.toLowerCase().indexOf(substring)
        const indexB = b.name.toLowerCase().indexOf(substring)
        return indexA - indexB || a.name.localeCompare(b.name)
      })
      .map((character): FilteredCharacters => {
        return {
          character,
          highlight: highlightAnalysis(inputText.value, character.name),
        }
      })
  } else {
    return [...characters.value]
      .sort((a, b) => a.name.localeCompare(b.name))
      .map((character): FilteredCharacters => {
        return { character, highlight: false }
      })
  }
})
function highlightAnalysis(needle: string, haystack: string): HighlightObject {
  const lowerName = haystack.toLowerCase()
  const lowerSub = needle.toLowerCase()
  const index = lowerName.indexOf(lowerSub)
  return {
    before: haystack.slice(0, index),
    match: haystack.slice(index, index + needle.length),
    after: haystack.slice(index + needle.length),
  }
}

function guessCharacter(id: string): void {
  counter.increment()
  if (id === correctAnswer) {
    win.value = true
  } else {
    incorrectAnswers.value.add(id)
  }
}

/*function hint(): void {
  cheatMode.value = true
  hintCounter.value += 1
}*/
</script>

<template>
  <main class="mt-10">
    <h3 class="pl-4 pb-4 text-xl text-purple-200 w-full flex justify-center">
      Which character is associated with the element
      <span class="text-green-200 ml-1">Californium?</span>
    </h3>
    <div class="flex justify-between items-center">
      <div class="flex gap-4 items-center">
        <input
          type="text"
          v-model="inputText"
          class="p-2 border rounded border-gray-700 ml-4 bg-gray-200 text-gray-800 outline-0"
          placeholder="Filter By Name"
        />
        You guessed {{ counter.count }} times
      </div>
      <div class="flex p-4 gap-2 items-center">
        <Button
          @click="cheatMode = !cheatMode"
          class="cursor-pointer bg-red-500/70 text-white p-2 rounded-2xl hover:bg-red-500/90 outline-0 transition-all border-0 shadow"
          >Hint</Button
        >
      </div>
    </div>
    <transition name="catalog" mode="out-in">
      <template v-if="filteredCharacters.length > 0">
        <div class="grid grid-cols-4 gap-4 p-4">
          <transition-group name="catalog">
            <div
              v-for="char in filteredCharacters"
              :key="char.character.id"
              class="card"
              :class="{
                wrong: incorrectAnswers.has(char.character.id),
                winner: correctAnswer === char.character.id && win,
              }"
              @click="guessCharacter(char.character.id)"
            >
              <img :src="char.character.avatar" class="mb-2 w-full" />

              <div>
                Name:
                <template v-if="char.highlight">
                  <span>{{ char.highlight.before }}</span>
                  <span class="bg-amber-600">{{ char.highlight.match }}</span>
                  <span>{{ char.highlight.after }}</span>
                </template>
                <template v-else>
                  {{ char.character.name }}
                </template>
              </div>
              <transition name="cheatmode">
                <div v-if="cheatMode">
                  Element: {{ char.character.affinity.name }} |
                  {{ char.character.affinity.symbol }} |
                  {{ char.character.affinity.atomicNumber }}
                </div>
              </transition>
            </div>
          </transition-group>
        </div>
      </template>
      <template v-else>
        <div class="p-4">{{ isFetching ? 'Fetching Data' : 'No Results Found' }}.</div>
      </template>
    </transition>
  </main>
</template>
<style scoped>
.catalog-enter-active,
.catalog-leave-active {
  transition: all 0.5s ease-in-out;
}
.catalog-enter-from,
.catalog-leave-to {
  opacity: 0;
  transform: translateX(30px);
}

.cheatmode-enter-active,
.cheatmode-leave-active {
  overflow: hidden;
  transition:
    max-height 0.5s ease,
    opacity 0.5s ease;
  max-height: 100px;
  opacity: 1;
}

.cheatmode-enter-from,
.cheatmode-leave-to {
  max-height: 0;
  opacity: 0;
}
</style>
