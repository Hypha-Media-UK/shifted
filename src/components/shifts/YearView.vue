<template>
  <div class="year-view">
    <div class="year-view__header">
      <button class="btn btn-secondary btn-icon" @click="previousYear">
        <span class="material-icons-round">chevron_left</span>
      </button>
      <h2 class="year-view__title">{{ currentYear }}</h2>
      <button class="btn btn-secondary btn-icon" @click="nextYear">
        <span class="material-icons-round">chevron_right</span>
      </button>
    </div>

    <div class="year-view__months">
      <MonthGrid
        v-for="month in months"
        :key="month.toISOString()"
        :month="month"
        :shifts="shifts"
        :accent-color="accentColor"
        @select-date="$emit('select-date', $event)"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
import { eachMonthOfInterval, startOfYear, endOfYear, addYears, subYears } from 'date-fns';
import MonthGrid from './MonthGrid.vue';

interface ShiftTime {
  startTime: string;
  endTime: string;
  isHoliday?: boolean;
}

interface Shift extends ShiftTime {
  id: number;
}

interface ShiftMap {
  [key: string]: Shift[];
}

defineProps<{
  shifts: ShiftMap;
  accentColor: string;
}>();

defineEmits<{
  (e: 'select-date', date: Date): void;
}>();

const currentDate = ref(new Date());
const currentYear = computed(() => currentDate.value.getFullYear());

const months = computed(() => {
  const start = startOfYear(currentDate.value);
  const end = endOfYear(currentDate.value);
  return eachMonthOfInterval({ start, end });
});

const nextYear = () => {
  currentDate.value = addYears(currentDate.value, 1);
};

const previousYear = () => {
  currentDate.value = subYears(currentDate.value, 1);
};
</script>

<style lang="scss" scoped>
@import '../../styles/abstracts/variables';

.year-view {
  &__header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: $spacing-lg;
    padding: $spacing-md $spacing-lg;
    background: white;
    border-radius: $border-radius-lg;
    box-shadow: $shadow-md;
    position: sticky;
    top: calc(73px + $spacing-xl); // Account for main header height
    z-index: 10;
    transition: box-shadow $transition-speed ease;

    @media (max-width: $breakpoint-sm) {
      top: calc(116px + $spacing-lg); // Adjusted for mobile header height
      padding: $spacing-sm;
      margin-bottom: $spacing-md;
    }

    &:hover {
      box-shadow: $shadow-lg;
    }
  }

  &__title {
    font-size: $font-size-xl;
    font-weight: $font-weight-bold;
    margin: 0;
    color: $text;
    text-align: center;
    min-width: 200px;

    @media (max-width: $breakpoint-sm) {
      font-size: $font-size-lg;
      min-width: auto;
    }
  }

  &__months {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
    gap: $spacing-lg;
    padding: $spacing-md;
    
    @media (max-width: $breakpoint-sm) {
      gap: $spacing-md;
      padding: $spacing-sm;
      grid-template-columns: 1fr;
    }
  }
}

.btn-icon {
  width: 40px;
  height: 40px;
  padding: 0;
  display: flex;
  align-items: center;
  justify-content: center;

  .material-icons-round {
    margin: 0;
    font-size: 1.25rem;
  }

  @media (max-width: $breakpoint-sm) {
    width: 32px;
    height: 32px;

    .material-icons-round {
      font-size: 1rem;
    }
  }
}
</style>
