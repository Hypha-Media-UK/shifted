<template>
  <div class="year-view">
    <div class="year-view__header">
      <button class="btn btn-secondary" @click="previousYear">
        ←
      </button>
      <h2 class="year-view__title">{{ currentYear }}</h2>
      <button class="btn btn-secondary" @click="nextYear">
        →
      </button>
    </div>

    <div class="year-view__grid">
      <MonthGrid
        v-for="month in months"
        :key="month.date"
        :month-date="new Date(month.date)"
        :days="month.days"
        :accent-color="accentColor"
        @day-click="handleDayClick"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
import {
  startOfYear,
  endOfYear,
  eachMonthOfInterval,
  getYear,
  addYears,
  subYears,
  startOfMonth,
  endOfMonth,
  eachDayOfInterval,
  format
} from 'date-fns';
import MonthGrid from './MonthGrid.vue';

interface ShiftTime {
  startTime: string;
  endTime: string;
}

interface Shift extends ShiftTime {
  id: number;
}

interface ShiftMap {
  [key: string]: Shift[];
}

const props = defineProps<{
  shifts: ShiftMap;
  accentColor: string;
}>();

const emit = defineEmits<{
  (e: 'select-date', date: Date): void;
}>();

const currentDate = ref(new Date());

const currentYear = computed(() => getYear(currentDate.value));

const months = computed(() => {
  const yearStart = startOfYear(currentDate.value);
  const yearEnd = endOfYear(currentDate.value);
  const monthsInYear = eachMonthOfInterval({ start: yearStart, end: yearEnd });

  return monthsInYear.map(monthDate => {
    const monthStart = startOfMonth(monthDate);
    const monthEnd = endOfMonth(monthDate);
    const daysInMonth = eachDayOfInterval({ start: monthStart, end: monthEnd });

    // Add empty days at the start for proper grid alignment
    const firstDayOfWeek = monthStart.getDay();
    const emptyDaysBefore = Array(firstDayOfWeek).fill(null).map((_, index) => ({
      date: new Date(monthStart).setDate(-index),
      inMonth: false,
      dayOfMonth: 0,
      shifts: []
    })).reverse();

    // Add empty days at the end for proper grid alignment
    const lastDayOfWeek = monthEnd.getDay();
    const emptyDaysAfter = Array(6 - lastDayOfWeek).fill(null).map((_, index) => ({
      date: new Date(monthEnd).setDate(monthEnd.getDate() + index + 1),
      inMonth: false,
      dayOfMonth: 0,
      shifts: []
    }));

    const days = [
      ...emptyDaysBefore,
      ...daysInMonth.map(date => ({
        date: date.getTime(),
        inMonth: true,
        dayOfMonth: date.getDate(),
        shifts: props.shifts[format(date, 'yyyy-MM-dd')] || []
      })),
      ...emptyDaysAfter
    ];

    return {
      date: monthDate.getTime(),
      days
    };
  });
});

const nextYear = () => {
  currentDate.value = addYears(currentDate.value, 1);
};

const previousYear = () => {
  currentDate.value = subYears(currentDate.value, 1);
};

const handleDayClick = (date: Date) => {
  emit('select-date', date);
};
</script>

<style lang="scss" scoped>
@import '@/styles/abstracts/variables';

.year-view {
  background: white;
  border-radius: $border-radius;
  padding: $spacing-md;
  box-shadow: $box-shadow;

  &__header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: $spacing-lg;
  }

  &__title {
    font-size: $font-size-lg;
    font-weight: 600;
    margin: 0;
    color: $text;
  }

  &__grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: $spacing-md;

    @media (min-width: $breakpoint-lg) {
      grid-template-columns: repeat(4, 1fr);
    }
  }
}
</style>
