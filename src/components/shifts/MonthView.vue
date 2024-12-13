<template>
  <div class="month-view">
    <div class="month-view__header">
      <button class="btn btn-secondary" @click="previousMonth">
        ←
      </button>
      <h2 class="month-view__title">{{ formattedMonth }}</h2>
      <button class="btn btn-secondary" @click="nextMonth">
        →
      </button>
    </div>

    <div class="month-view__days">
      <DayRow
        v-for="date in daysInMonth"
        :key="date.toISOString()"
        :date="date"
        :shifts="getShiftsForDate(date)"
        :accent-color="accentColor"
        :clipboard-shift="clipboardShift"
        :is-expanded="expandedDate === date.toISOString()"
        @update-shifts="(shifts) => updateShiftsForDate(date, shifts)"
        @copy-shift="setClipboardShift"
        @clear-clipboard="clearClipboardShift"
        @expand="handleExpand(date)"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
import { 
  startOfMonth, 
  endOfMonth, 
  eachDayOfInterval, 
  format,
  addMonths,
  subMonths
} from 'date-fns';
import DayRow from './DayRow.vue';

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
  accentColor: string;
}>();

const emit = defineEmits<{
  (e: 'update-shifts', shifts: ShiftMap): void;
}>();

const currentMonth = ref(new Date());
const clipboardShift = ref<ShiftTime | null>(null);
const shifts = ref<ShiftMap>({});
const expandedDate = ref<string | null>(null);

// Computed properties
const formattedMonth = computed(() => format(currentMonth.value, 'MMMM yyyy'));

const daysInMonth = computed(() => {
  const start = startOfMonth(currentMonth.value);
  const end = endOfMonth(currentMonth.value);
  return eachDayOfInterval({ start, end });
});

// Methods
const getShiftsForDate = (date: Date): Shift[] => {
  const dateKey = format(date, 'yyyy-MM-dd');
  return shifts.value[dateKey] || [];
};

const updateShiftsForDate = (date: Date, newShifts: Shift[]) => {
  const dateKey = format(date, 'yyyy-MM-dd');
  shifts.value = {
    ...shifts.value,
    [dateKey]: newShifts
  };
  emit('update-shifts', shifts.value);
};

const setClipboardShift = (shift: ShiftTime) => {
  clipboardShift.value = shift;
};

const clearClipboardShift = () => {
  clipboardShift.value = null;
};

const handleExpand = (date: Date) => {
  const dateString = date.toISOString();
  expandedDate.value = expandedDate.value === dateString ? null : dateString;
};

const nextMonth = () => {
  currentMonth.value = addMonths(currentMonth.value, 1);
  expandedDate.value = null;
};

const previousMonth = () => {
  currentMonth.value = subMonths(currentMonth.value, 1);
  expandedDate.value = null;
};
</script>

<style lang="scss" scoped>
@import '@/styles/abstracts/variables';

.month-view {
  &__header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: $spacing-lg;
    padding: $spacing-md;
    background: white;
    border-radius: $border-radius;
    box-shadow: $box-shadow;
  }

  &__title {
    font-size: $font-size-lg;
    font-weight: 600;
    margin: 0;
    color: $text;
  }

  &__days {
    display: flex;
    flex-direction: column;
    gap: $spacing-sm;
  }
}
</style>
