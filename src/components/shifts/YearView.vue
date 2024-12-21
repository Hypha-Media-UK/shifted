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
  isHoliday?: boolean;
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
  currentDate: Date;
}>();

defineEmits<{
  (e: 'select-date', date: Date): void;
}>();

const months = computed(() => {
  const start = startOfYear(props.currentDate);
  const end = endOfYear(props.currentDate);
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
