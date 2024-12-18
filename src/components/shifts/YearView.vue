<template>
  <div class="year-view">
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
import { computed } from 'vue';
import { eachMonthOfInterval, startOfYear, endOfYear } from 'date-fns';
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

defineProps<{
  shifts: ShiftMap;
  accentColor: string;
}>();

defineEmits<{
  (e: 'select-date', date: Date): void;
}>();

const months = computed(() => {
  const now = new Date();
  const start = startOfYear(now);
  const end = endOfYear(now);
  return eachMonthOfInterval({ start, end });
});
</script>

<style lang="scss" scoped>
@import '../../styles/abstracts/variables';

.year-view {
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
</style>
