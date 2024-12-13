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
      <div 
        v-for="month in months" 
        :key="month.date"
        class="year-view__month"
      >
        <h3 class="year-view__month-title">{{ month.name }}</h3>
        <div class="year-view__month-grid">
          <div 
            v-for="day in month.days" 
            :key="day.date"
            class="year-view__day"
            :class="{
              'is-empty': !day.inMonth,
              'has-shifts': day.shifts.length > 0
            }"
            :style="day.shifts.length ? { backgroundColor: accentColor + '33' } : {}"
          >
            <span v-if="day.inMonth">{{ day.dayOfMonth }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import {
  startOfYear,
  endOfYear,
  eachMonthOfInterval,
  format,
  getYear,
  addYears,
  subYears,
  startOfMonth,
  endOfMonth,
  eachDayOfInterval,
  isSameMonth,
  getDate
} from 'date-fns';

const props = defineProps({
  shifts: {
    type: Object,
    default: () => ({})
  },
  accentColor: {
    type: String,
    default: '#4A90E2'
  }
});

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
      dayOfMonth: '',
      shifts: []
    })).reverse();

    // Add empty days at the end for proper grid alignment
    const lastDayOfWeek = monthEnd.getDay();
    const emptyDaysAfter = Array(6 - lastDayOfWeek).fill(null).map((_, index) => ({
      date: new Date(monthEnd).setDate(monthEnd.getDate() + index + 1),
      inMonth: false,
      dayOfMonth: '',
      shifts: []
    }));

    const days = [
      ...emptyDaysBefore,
      ...daysInMonth.map(date => ({
        date: date.getTime(),
        inMonth: true,
        dayOfMonth: getDate(date),
        shifts: props.shifts[format(date, 'yyyy-MM-dd')] || []
      })),
      ...emptyDaysAfter
    ];

    return {
      date: monthDate.getTime(),
      name: format(monthDate, 'MMM'),
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

  &__month {
    &-title {
      font-size: $font-size-sm;
      font-weight: 500;
      margin-bottom: $spacing-sm;
      color: rgba($text, 0.8);
    }

    &-grid {
      display: grid;
      grid-template-columns: repeat(7, 1fr);
      gap: 2px;
    }
  }

  &__day {
    aspect-ratio: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 0.75rem;
    border-radius: 2px;
    background-color: $background;
    color: $text;
    transition: background-color $transition-speed ease;

    &.is-empty {
      background-color: transparent;
    }

    &.has-shifts {
      font-weight: 500;
    }
  }
}
</style>
