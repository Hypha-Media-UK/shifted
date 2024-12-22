<template>
  <div class="month-view">
    <div class="month-view__days">
      <DayRow
        v-for="date in daysInMonth"
        :key="date.toISOString()"
        :date="date"
        :shifts="getShiftsForDate(date)"
        :accent-color="accentColor"
        :clipboard-shift="props.clipboardShift"
        :is-expanded="expandedDate === date.toISOString()"
        :is-selected="!!selectedDate && isSameDay(date, selectedDate)"
        :ref="(el) => {
          if (!!selectedDate && isSameDay(date, selectedDate)) {
            selectedDayRef = el;
          }
        }"
        @update-shifts="(shifts) => updateShiftsForDate(date, shifts)"
        @copy-shift="$emit('copy-shift', $event)"
        @clear-clipboard="$emit('clear-clipboard')"
        @expand="() => emit('edit-shift', { date })"
        @edit="(shift) => emit('edit-shift', { shift, date })"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch, nextTick } from 'vue';
import { 
  startOfMonth, 
  endOfMonth, 
  eachDayOfInterval, 
  format,
  isSameMonth,
  isSameDay,
  startOfDay
} from 'date-fns';
import DayRow from './DayRow.vue';

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
  accentColor: string;
  selectedDate?: Date;
  initialShifts?: ShiftMap;
  currentDate: Date;
  clipboardShift: ShiftTime | null;
}>();

const emit = defineEmits<{
  (e: 'update-shifts', shifts: ShiftMap): void;
  (e: 'clear-selected-date'): void;
  (e: 'edit-shift', data: { shift?: Shift, date: Date }): void;
  (e: 'copy-shift', shift: ShiftTime): void;
  (e: 'clear-clipboard'): void;
}>();

const shifts = ref<ShiftMap>({});
const expandedDate = ref<string | null>(null);
const selectedDayRef = ref<any>(null);

// Watch for selectedDate changes
watch(() => props.selectedDate, async (newDate) => {
  if (newDate) {
    // Wait for the DOM to update
    await nextTick();
    
    // Scroll the selected day into view with smooth behavior
    if (selectedDayRef.value?.$el) {
      selectedDayRef.value.$el.scrollIntoView({
        behavior: 'smooth',
        block: 'center'
      });
    }
  }
}, { immediate: true });

// Watch for initialShifts changes
watch(() => props.initialShifts, (newShifts) => {
  if (newShifts) {
    shifts.value = newShifts;
  }
}, { immediate: true });

const daysInMonth = computed(() => {
  const start = startOfMonth(props.currentDate);
  const end = endOfMonth(props.currentDate);
  return eachDayOfInterval({ start, end });
});

const getShiftsForDate = (date: Date): Shift[] => {
  const dateKey = format(date, 'yyyy-MM-dd');
  return shifts.value[dateKey] || [];
};

const updateShiftsForDate = (date: Date, newShifts: Shift[]) => {
  const dateKey = format(date, 'yyyy-MM-dd');
  const updatedShifts = {
    ...shifts.value,
    [dateKey]: newShifts
  };
  shifts.value = updatedShifts;
  emit('update-shifts', updatedShifts);
};
</script>

<style lang="scss" scoped>
@import '../../styles/abstracts/variables';

.month-view {
  &__days {
    display: flex;
    flex-direction: column;
    gap: $spacing-sm;
    padding-bottom: $spacing-xl;

    @media (max-width: $breakpoint-sm) {
      gap: $spacing-xs;
      padding-bottom: $spacing-lg;
    }
  }
}
</style>
