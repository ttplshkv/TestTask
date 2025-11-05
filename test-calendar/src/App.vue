<script setup>
import { ref } from 'vue'
import Calendar from './components/Calendar.vue'

const locale = ref('ru-RU')
const initial = ref('')
const selected = ref('')

function onDayClick(dateIso) {
  selected.value = dateIso
}
</script>

<template>
  <div style="padding:24px">
    <div style="display:flex;gap:8px;align-items:center;margin-bottom:12px;">
      <label>Язык:
        <select v-model="locale">
          <option value="ru-RU">ru-RU</option>
          <option value="en-US">en-US</option>
          <option value="de-DE">de-DE</option>
        </select>
      </label>

      <label>Начальная дата:
        <input v-model="initial" placeholder="2025-11-05" />
      </label>
    </div>

    <Calendar 
      :initial-date="initial || undefined"
      :locale="locale"
      @day-click="onDayClick" 
    />

    <p style="margin-top:12px">Выбрано: <b>{{ selected }}</b></p>
  </div>
</template>
