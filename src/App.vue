<template>
  <main :class="['page', { dark: darkMode }]">
    <section class="card">
      <div class="top-row">
        <div>
          <h1>Medium Number Finder</h1>
          <p>Enter two numbers or use the sliders, then find the single midpoint once.</p>
        </div>
        <button type="button" class="toggle-button" @click="toggleDarkMode">
          {{ darkMode ? 'Light mode' : 'Dark mode' }}
        </button>
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

      <button @click="calculate" class="action">Find medium number</button>

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
      </section>

      <p v-else class="hint">Type two numbers and click the button to calculate.</p>
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

function toggleDarkMode() {
  darkMode.value = !darkMode.value
}

function startDrag(event) {
  if (!steps.value.length) return
  event.dataTransfer.setData('text/plain', String(steps.value[0].mid))
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
}
</script>
