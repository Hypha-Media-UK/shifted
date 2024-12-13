<template>
  <div class="month-grid">
    <h3 class="month-grid__title">{{ monthName }}</h3>
    <div class="month-grid__days">
      <div 
        v-for="day in days" 
        :key="day.date"
        class="month-grid__day"
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
</template>

<script setup lang="ts">
import { computed } from 'vue';
import { format } from 'date-fns';

interface Day {
  date: number;
  inMonth: boolean;
  dayOfMonth: number;
  shifts: any[];
}

const props = defineProps<{
  monthDate: Date;
  days: Day[];
  accentColor: string;
}>();

const monthName = computed(() => format(props.monthDate, 'MMM'));
</script>

<style lang="scss" scoped>
@import '@/styles/abstracts/variables';

.month-grid {
  &__title {
    font-size: $font-size-sm;
    font-weight: 500;
    margin-bottom: $spacing-sm;
    color: rgba($text, 0.8);
  }

  &__days {
    display: grid;
    grid-template-columns: repeat(7, 1fr);
    gap: 2px;
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
