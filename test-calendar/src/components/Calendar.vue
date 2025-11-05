<template>

  <div style="width: 300px; border: 1px solid #ccc; border-radius: 2px; padding: 8px; box-sizing: border-box; overflow: hidden;">
    <!-- Шапка: стрелки и подпись месяца -->
    <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 8px;">

      <button @click="prevMonth" aria-label="Предыдущий месяц" style="width: 24px; height: 24px; 
                                                                background: #fff; cursor: pointer; 
                                                                display: flex; align-items: center; 
                                                                justify-content: center; line-height: 1; 
                                                                padding: 0;">
                                                                ◀
                                                                </button>
      <div style="font-weight: 500; text-transform: capitalize;">{{ monthLabel }}</div>
      <button @click="nextMonth" aria-label="Следующий месяц" style="width: 24px; height: 24px;
                                                                background: #fff; cursor: pointer;
                                                                display: flex; align-items: center; 
                                                                justify-content: center; line-height: 1; 
                                                                padding:0;">
                                                                ▶
                                                                </button>

    </div>

    <!-- Заголовок дней недели -->
    <div style="display: grid; grid-template-columns: repeat(7,1fr); column-gap: 4px; row-gap: 8px; margin-bottom: 6px;">
      <div v-for="(w,i) in weekdays" :key="i" style="text-align: center; font-size: 12px; color: #6b7280; min-width: 0;">
        {{ w }}
      </div>
    </div>

    <!-- Сетка дней -->
    <div style="display: grid; grid-template-columns: repeat(7,1fr); row-gap: 2px; min-width: 0;">
      <!-- Пустые ячейки перед первым числом -->
      <div v-for="n in leadingEmpty" :key="'e'+n"></div>

      <!-- Дни месяца -->
      <button
        v-for="day in daysInMonth"
        :key="day"
        @click="selectDay(day)"
        :style="dayStyle(day)"
      >
        {{ day }}
      </button>
    </div>

  </div>
</template>

<script setup>
import { ref, onMounted, watch, computed } from 'vue'

     const props = defineProps({
        initialDate: { 
            type: String, 
            default: '', 
            validator: (v) => v === '' || /^[0-9]{4}-[0-9]{2}-[0-9]{2}$/.test(v)
        },
        locale: { type: String, default: 'ru-RU'}
     });


     const emit = defineEmits(['day-click']);
     const today = new Date();
     const selected = ref(null);
     const viewYear = ref(today.getFullYear());
     const viewMonth = ref(today.getMonth());


     function parseIso(s) {

        const m = /^([0-9]{4})-([0-9]{2})-([0-9]{2})$/.exec(s);
        if (!m) return null;

        const year = +m[1];
        const month = +m[2];
        const day = +m[3];

        const full_date = new Date(year, month - 1, day);

        if (full_date.getFullYear() !== year || 
        full_date.getMonth() !== (month - 1) || 
        full_date.getDate() !== day) {
            return null;
        }
        return full_date;
     }

    
     onMounted(() => {
        if (props.initialDate) {
            const date = parseIso(props.initialDate);
            
            if (date) {
                selected.value = date;
                viewYear.value = date.getFullYear();
                viewMonth.value = date.getMonth();
                return;
            }
        }
        selected.value = new Date(today.getFullYear(), today.getMonth(), today.getDate());
     })

     watch(() => props.initialDate, (val) => {
        if (!val) return;

        const date = parseIso(val);
        if (date) {
            selected.value = date;
            viewYear.value = date.getFullYear();
            viewMonth.value = date.getMonth();
        }
     })

    const monthLabel = computed(() => {
        const fmt = new Intl.DateTimeFormat(props.locale, { month: 'long', year: 'numeric' });
        const s = fmt.format(new Date(viewYear.value, viewMonth.value, 1));
        return s.charAt(0).toUpperCase() + s.slice(1);
     });

    const weekdays = computed(() => {
        // Воскресенье первым: 1 Aug 2021 — воскресенье
        const base = new Date(2021, 7, 1);
        const fmt = new Intl.DateTimeFormat(props.locale, { weekday: 'short' });

        return Array.from({ length: 7 }, (_, i) =>
            fmt.format(new Date(base.getFullYear(), base.getMonth(), base.getDate() + i)).replace('.', '')
        );

    });

    const daysInMonth = computed(() =>
         new Date(viewYear.value, viewMonth.value + 1, 0).getDate()
    );

    const leadingEmpty = computed(() => {
        // Неделя начинается с воскресенья, getDay() уже даёт 0-вс
        return new Date(viewYear.value, viewMonth.value, 1).getDay();
    });

    function prevMonth() {
        const date = new Date(viewYear.value, viewMonth.value - 1, 1);
        viewYear.value = date.getFullYear();
        viewMonth.value = date.getMonth();
    }

    function nextMonth() {
        const date = new Date(viewYear.value, viewMonth.value + 1, 1);
        viewYear.value = date.getFullYear();
        viewMonth.value = date.getMonth();
    }

    function toIso(d) {
        const yyyy = d.getFullYear();
        const mm = String(d.getMonth() + 1).padStart(2, '0');
        const dd = String(d.getDate()).padStart(2, '0');
        return `${yyyy}-${mm}-${dd}`;
    }

    function isToday(y, m, d) {
        return today.getFullYear() === y && today.getMonth() === m && today.getDate() === d;
    }

    function isSelected(y, m, d) {
        if (!selected.value) return false;

        return selected.value.getFullYear() === y &&
                selected.value.getMonth() === m &&
                selected.value.getDate() === d;
    }

    function selectDay(day) {
        const d = new Date(viewYear.value, viewMonth.value, day);
        selected.value = d;
        emit('day-click', toIso(d));
    }

    function dayStyle(day) {
        const base = 'height: 30px; width: 100%; border: 1px solid transparent; border-radius: 2px; background: #fff; cursor: pointer; box-sizing: border-box; min-width: 0;';
        const hover = 'transition: background .1s;';
        const pad = 'display: flex; align-items: center; justify-content: center;';
        const y = viewYear.value, m = viewMonth.value, d = day;

        if (isSelected(y, m, d)) {
            return base + pad + hover + 'border-color: #3b82f6; background: #fff; color: #111827;';
        }

        return base + pad + hover + 'background: #fff;';
    }

</script>