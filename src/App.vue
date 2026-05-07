<template>
  <main :class="['page', { dark: darkMode }]">
    <section class="card">
      <div class="top-row">
        <div>
          <h1>Medium Number Finder</h1>
          <p>Enter two numbers or use the sliders, then find the single midpoint once.</p>
        </div>
        <div class="top-controls">
          <button type="button" class="toggle-button" @click="toggleDarkMode">
            {{ darkMode ? 'Light mode' : 'Dark mode' }}
          </button>
          <button type="button" class="toggle-button" @click="toggleGuessGame">
            {{ showGuessGame ? 'Hide Game' : 'Show Game' }}
          </button>
        </div>
      </div>

      <div class="range-values">
        <div class="range-group">
          <div class="range-label"><span>First:</span></div>
          <input
            class="direct-input"
            type="number"
            min="1"
            :max="second - 1"
            :value="first"
            @input="updateFirst"
            @dragover.prevent
            @drop="handleFirstDrop"
          />
          <div class="range-buttons">
            <button type="button" @click="shiftFirst(-10)">-10</button>
            <button type="button" @click="shiftFirst(-5)">-5</button>
            <button type="button" @click="shiftFirst(-1)">-1</button>
            <button type="button" @click="shiftFirst(1)">+1</button>
            <button type="button" @click="shiftFirst(5)">+5</button>
            <button type="button" @click="shiftFirst(10)">+10</button>
            <button type="button" @click="resetFirst" class="reset-btn">Set to 1</button>
          </div>
        </div>

        <div class="range-group">
          <div class="range-label"><span>Second:</span></div>
          <input
            class="direct-input"
            type="number"
            :min="first + 1"
            max="9999"
            :value="second"
            @input="updateSecond"
            @dragover.prevent
            @drop="handleSecondDrop"
          />
          <div class="range-buttons">
            <button type="button" @click="shiftSecond(-10)">-10</button>
            <button type="button" @click="shiftSecond(-5)">-5</button>
            <button type="button" @click="shiftSecond(-1)">-1</button>
            <button type="button" @click="shiftSecond(1)">+1</button>
            <button type="button" @click="shiftSecond(5)">+5</button>
            <button type="button" @click="shiftSecond(10)">+10</button>
            <button type="button" @click="resetSecond" class="reset-btn">Set to 9999</button>
          </div>
        </div>
      </div>

      <div class="range-slider">
        <div class="slider-track" :style="trackStyle"></div>
        <input
          class="range-input range-left"
          type="range"
          min="1"
          max="9999"
          step="1"
          :value="first"
          @input="updateFirst"
        />
        <input
          class="range-input range-right"
          type="range"
          min="1"
          max="9999"
          step="1"
          :value="second"
          @input="updateSecond"
        />
      </div>

      <div class="action-buttons">
        <button @click="calculate" class="action">Find medium number</button>
        <button @click="randomPick" class="action secondary">Random pick</button>
      </div>

      <section v-if="steps.length" class="result">
        <p class="single-step">
          between {{ steps[0].left }} and {{ steps[0].right }} is
          <strong
            class="draggable-result"
            draggable="true"
            @dragstart="startDrag"
          >{{ steps[0].mid }}</strong>
        </p>
        <p class="hint small">Drag the result into either input to replace that value.</p>
        <div class="result-actions">
          <button type="button" class="action" @click="replaceFirstWithMid">Set first to result</button>
          <button type="button" class="action secondary" @click="replaceSecondWithMid">Set second to result</button>
        </div>
      </section>

      <section v-if="randomResult !== null" class="result">
        <p class="single-step">
          Random number between {{ first }} and {{ second }} is
          <strong
            class="draggable-result"
            draggable="true"
            @dragstart="startRandomDrag"
          >{{ randomResult }}</strong>
        </p>
        <p class="hint small">Drag the random result into either input to replace that value.</p>
        <div class="result-actions">
          <button type="button" class="action" @click="replaceFirstWithMid">Set first to result</button>
          <button type="button" class="action secondary" @click="replaceSecondWithMid">Set second to result</button>
        </div>
      </section>

      <section v-if="showGuessGame" class="guess-section">
        <h2>Guess the Number</h2>
        <div class="current-range" :class="{ 'range-low': lastGuessResult === 'low', 'range-high': lastGuessResult === 'high', 'range-correct': lastGuessResult === 'correct' }">
          <p><strong>Current Range:</strong> {{ guessRangeMin }} - {{ guessRangeMax }}</p>
        </div>

        <div v-if="!gameActive" class="game-start">
          <button @click="startGame" class="action">Start New Game</button>
        </div>

        <div v-if="gameActive || feedback" class="game-active">

          <div v-if="gameActive" class="guess-input">
            <input
              type="number"
              v-model.number="guess"
              :min="guessRangeMin"
              :max="guessRangeMax"
              placeholder="Enter your guess"
              @keyup.enter="makeGuess"
            />
            <button @click="makeGuess" class="action">Guess</button>
          </div>

          <div v-if="gameActive" class="game-buttons">
            <button @click="revealSecret" class="action secondary">Reveal Secret</button>
            <button @click="endGame" class="action secondary">End Game</button>
          </div>
        </div>
      </section>
    </section>
  </main>
</template>

<script setup>
import { computed, ref } from 'vue'

const first = ref(1)
const second = ref(9999)
const steps = ref([])
const minValue = 1
const maxValue = 9999
const darkMode = ref(false)
const randomResult = ref(null)
const showGuessGame = ref(false)
const gameActive = ref(false)
const targetNumber = ref(null)
const guess = ref('')
const guessCount = ref(0)
const feedback = ref('')
const feedbackClass = ref('')
const guessHistory = ref([])
const guessRangeMin = ref(1)
const guessRangeMax = ref(9999)
const lastGuessResult = ref('') // 'low', 'high', or ''

const trackStyle = computed(() => {
  const leftPercent = ((first.value - minValue) / (maxValue - minValue)) * 100
  const rightPercent = ((second.value - minValue) / (maxValue - minValue)) * 100

  return {
    background: `linear-gradient(to right, #e2e8f0 ${leftPercent}%, #2563eb ${leftPercent}%, #2563eb ${rightPercent}%, #e2e8f0 ${rightPercent}%)`
  }
})

function updateFirst(event) {
  const value = Number(event.target.value)
  if (Number.isNaN(value)) return
  first.value = Math.min(Math.max(value, minValue), second.value - 1)
}

function updateSecond(event) {
  const value = Number(event.target.value)
  if (Number.isNaN(value)) return
  second.value = Math.max(Math.min(value, maxValue), first.value + 1)
}

function shiftFirst(amount) {
  first.value = Math.max(minValue, Math.min(first.value + amount, second.value - 1))
}

function shiftSecond(amount) {
  second.value = Math.min(maxValue, Math.max(second.value + amount, first.value + 1))
}

function resetFirst() {
  first.value = minValue
}

function resetSecond() {
  second.value = maxValue
}

function toggleDarkMode() {
  darkMode.value = !darkMode.value
}

function toggleGuessGame() {
  showGuessGame.value = !showGuessGame.value
  if (!showGuessGame.value) {
    endGame()
  }
}

function replaceFirstWithMid() {
  const value = steps.value.length ? steps.value[0].mid : randomResult.value
  if (value === null || value === undefined) return
  first.value = Math.min(Math.max(value, minValue), second.value - 1)
}

function replaceSecondWithMid() {
  const value = steps.value.length ? steps.value[0].mid : randomResult.value
  if (value === null || value === undefined) return
  second.value = Math.max(Math.min(value, maxValue), first.value + 1)
}

function startDrag(event) {
  if (!steps.value.length) return
  event.dataTransfer.setData('text/plain', String(steps.value[0].mid))
  event.dataTransfer.effectAllowed = 'copy'
}

function startRandomDrag(event) {
  if (randomResult.value === null) return
  event.dataTransfer.setData('text/plain', String(randomResult.value))
  event.dataTransfer.effectAllowed = 'copy'
}

function handleFirstDrop(event) {
  const value = Number(event.dataTransfer.getData('text/plain'))
  if (Number.isNaN(value)) return
  first.value = Math.min(Math.max(value, minValue), second.value - 1)
}

function handleSecondDrop(event) {
  const value = Number(event.dataTransfer.getData('text/plain'))
  if (Number.isNaN(value)) return
  second.value = Math.max(Math.min(value, maxValue), first.value + 1)
}

function calculate() {
  let left = Number(first.value)
  let right = Number(second.value)

  if (left > right) {
    ;[left, right] = [right, left]
  }

  const mid = Math.round((left + right) / 2)
  steps.value = [{ left, right, mid }]
  randomResult.value = null
}

function randomPick() {
  const min = Math.min(first.value, second.value)
  const max = Math.max(first.value, second.value)
  randomResult.value = Math.floor(Math.random() * (max - min + 1)) + min
  steps.value = []
}

function startGame() {
  guessRangeMin.value = 1
  guessRangeMax.value = 9999

  const min = Math.min(first.value, second.value)
  const max = Math.max(first.value, second.value)
  targetNumber.value = Math.floor(Math.random() * (max - min + 1)) + min
  gameActive.value = true
  guess.value = ''
  guessCount.value = 0
  feedback.value = ''
  feedbackClass.value = ''
  guessHistory.value = []
  guessRangeMin.value = min
  guessRangeMax.value = max
  lastGuessResult.value = ''
}

function makeGuess() {
  if (!gameActive.value || guess.value === '') return

  const userGuess = Number(guess.value)
  guessCount.value++
  guessHistory.value.push(userGuess)

  if (userGuess < targetNumber.value) {
    feedback.value = 'Too low! Try a higher number.'
    feedbackClass.value = 'low'
    lastGuessResult.value = 'low'
    guessRangeMin.value = userGuess
  } else if (userGuess > targetNumber.value) {
    feedback.value = 'Too high! Try a lower number.'
    feedbackClass.value = 'high'
    lastGuessResult.value = 'high'
    guessRangeMax.value = userGuess
  } else {
    feedback.value = `Correct! You guessed it in ${guessCount.value} tries.`
    feedbackClass.value = 'correct'
    gameActive.value = false
    lastGuessResult.value = 'correct'
    guessRangeMin.value = userGuess
    guessRangeMax.value = userGuess
  }

  guess.value = ''
}

function revealSecret() {
  if (!gameActive.value) return
  feedback.value = `The secret number was ${targetNumber.value}.`
  feedbackClass.value = 'reveal'
  gameActive.value = false
  lastGuessResult.value = 'correct'
  guessRangeMin.value = targetNumber.value
  guessRangeMax.value = targetNumber.value
}

function endGame() {
  gameActive.value = false
  targetNumber.value = null
  guess.value = ''
  guessCount.value = 0
  feedback.value = ''
  feedbackClass.value = ''
  lastGuessResult.value = ''
  guessRangeMin.value = 1
  guessRangeMax.value = 9999
}
</script>
