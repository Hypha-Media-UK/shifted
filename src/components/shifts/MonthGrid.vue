<template>
  <div class="month-grid">
    <h3 class="month-grid__title">{{ monthName }}</h3>
    <div class="month-grid__days">
      <div 
        v-for="day in daysInMonth" 
        :key="day.date"
        class="month-grid__day"
        :class="{
          'is-empty': !day.inMonth,
          'has-shifts': day.shifts.length > 0,
          'is-clickable': day.inMonth && day.shifts.length > 0
        }"
        :style="day.shifts.length ? { backgroundColor: accentColor + '15' } : {}"
        @click="handleDayClick(day)"
      >
        <span v-if="day.inMonth">{{ day.dayOfMonth }}</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue';
import { 
  format, 
  startOfMonth, 
  endOfMonth, 
  eachDayOfInterval, 
  startOfWeek,
  endOfWeek,
  isSameMonth,
  getDate
} from 'date-fns';

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

interface Day {
  date: number;
  inMonth: boolean;
  dayOfMonth: number;
  shifts: Shift[];
}

const props = defineProps<{
  month: Date;
  shifts: ShiftMap;
  accentColor: string;
}>();

const emit = defineEmits<{
  (e: 'select-date', date: Date): void;
}>();

const monthName = computed(() => format(props.month, 'MMMM'));

const daysInMonth = computed(() => {
  const start = startOfWeek(startOfMonth(props.month));
  const end = endOfWeek(endOfMonth(props.month));
  
  return eachDayOfInterval({ start, end }).map(date => {
    const dateKey = format(date, 'yyyy-MM-dd');
    return {
      date: date.getTime(),
      inMonth: isSameMonth(date, props.month),
      dayOfMonth: getDate(date),
      shifts: props.shifts[dateKey] || []
    };
  });
});

const handleDayClick = (day: Day) => {
  if (day.inMonth && day.shifts.length > 0) {
    emit('select-date', new Date(day.date));
  }
};
</script>

<style lang="scss" scoped>
@import '../../styles/abstracts/variables';

.month-grid {
  background: white;
  border-radius: $border-radius-lg;
  padding: $spacing-lg;
  box-shadow: $shadow-sm;
  transition: all $transition-speed ease;

  &:hover {
    box-shadow: $shadow-md;
    transform: translateY(-2px);
  }

  &__title {
    font-size: $font-size-lg;
    font-weight: $font-weight-bold;
    margin-bottom: $spacing-md;
    color: $text;
    text-align: center;
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
    font-size: $font-size-sm;
    border-radius: $border-radius-sm;
    background-color: $background;
    color: $text;
    transition: all $transition-speed $transition-bounce;
    border: 1px solid transparent;

    &.is-empty {
      background-color: transparent;
    }

    &.has-shifts {
      font-weight: $font-weight-bold;
      border-color: rgba($primary, 0.1);
    }

    &.is-clickable {
      cursor: pointer;

      &:hover {
        transform: scale(1.1);
        box-shadow: $shadow-md;
        z-index: 1;
      }
    }

    @media (max-width: $breakpoint-sm) {
      font-size: $font-size-sm;
    }
  }
}
</style>
