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
        :style="getDayStyle(day)"
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
  isHoliday?: boolean;
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

const getTimeOfDay = (shift: ShiftTime) => {
  if (shift.isHoliday) {
    return 'holiday';
  }

  const hour = parseInt(shift.startTime.split(':')[0]);
  
  if (hour >= 5 && hour < 12) {
    return 'morning';
  } else if (hour >= 12 && hour < 17) {
    return 'afternoon';
  } else if (hour >= 17 && hour < 22) {
    return 'evening';
  } else {
    return 'night';
  }
};

const getDayStyle = (day: Day) => {
  if (!day.shifts.length) return {};

  // Get the first shift's time period to determine the color
  const firstShift = day.shifts[0];
  const timeOfDay = getTimeOfDay(firstShift);
  
  const colors = {
    morning: 'var(--morning-color)',
    afternoon: 'var(--afternoon-color)',
    evening: 'var(--evening-color)',
    night: 'var(--night-color)',
    holiday: 'var(--holiday-color)'
  };

  const borderColors = {
    morning: 'var(--morning-border)',
    afternoon: 'var(--afternoon-border)',
    evening: 'var(--evening-border)',
    night: 'var(--night-border)',
    holiday: 'var(--holiday-border)'
  };

  return {
    '--shift-color': colors[timeOfDay],
    '--shift-border': borderColors[timeOfDay]
  };
};

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
  border-radius: $border-radius;
  padding: $spacing-lg;
  box-shadow: $shadow-sm;
  transition: box-shadow $transition-speed ease;

  &:hover {
    box-shadow: $shadow-md;
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
    --morning-color: #{rgba($morning, 0.15)};
    --afternoon-color: #{rgba($afternoon, 0.15)};
    --evening-color: #{rgba($evening, 0.15)};
    --night-color: #{rgba($night, 0.15)};
    --holiday-color: #{rgba($holiday, 0.15)};
    --morning-border: #{rgba($morning, 0.4)};
    --afternoon-border: #{rgba($afternoon, 0.4)};
    --evening-border: #{rgba($evening, 0.4)};
    --night-border: #{rgba($night, 0.4)};
    --holiday-border: #{rgba($holiday, 0.4)};
    --shift-color: transparent;
    --shift-border: transparent;

    aspect-ratio: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: $font-size-sm;
    border-radius: $border-radius-sm;
    background-color: $background;
    color: $text;
    transition: all $transition-speed ease;
    border: 1px solid transparent;

    &.is-empty {
      background-color: transparent;
    }

    &.has-shifts {
      font-weight: $font-weight-bold;
      background-color: var(--shift-color);
      border-color: var(--shift-border);
    }

    &.is-clickable {
      cursor: pointer;

      &:hover {
        transform: scale(1.05);
        box-shadow: $shadow-sm;
        background-color: var(--shift-color);
        border-color: var(--shift-border);
      }
    }

    @media (max-width: $breakpoint-sm) {
      font-size: $font-size-sm;
    }
  }
}
</style>
