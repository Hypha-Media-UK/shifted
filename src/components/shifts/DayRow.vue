<template>
  <div 
    class="day-row" 
    :class="{ 
      'is-expanded': isExpanded,
      'is-selected': isSelected,
      'is-past': isPastDay
    }"
  >
    <div class="day-row__header" @click="!isPastDay && $emit('expand')">
      <div class="day-row__date">
        <span class="day-row__weekday">{{ formattedWeekday }}</span>
        <span class="day-row__day">{{ formattedDay }}</span>
      </div>
      <div class="day-row__shifts">
        <template v-if="shifts.length">
          <ShiftPill
            v-for="shift in sortedShifts"
            :key="shift.id"
            :shift="shift"
            :accent-color="accentColor"
            :is-past="isPastDay"
            @edit="$emit('edit', shift)"
            @delete="quickDelete(shift.id)"
          />
        </template>
        <div v-else class="day-row__empty" :class="{ 'is-past': isPastDay }">
          <span class="material-icons-round">schedule</span>
          No shifts
        </div>
        <div class="day-row__actions">
          <button 
            v-if="clipboardShift"
            class="day-row__action-btn day-row__add-btn"
            :class="{ 'is-disabled': shifts.length >= 3 || hasClipboardShift || wouldClipboardShiftOverlap || isPastDay }"
            @click.stop="applyClipboardShift"
            :disabled="shifts.length >= 3 || hasClipboardShift || wouldClipboardShiftOverlap || isPastDay"
            :title="getAddButtonTitle"
          >
            <span class="material-icons-round">content_paste</span>
          </button>
          <button 
            class="day-row__action-btn day-row__new-btn"
            :class="{ 'is-disabled': shifts.length >= 3 || isPastDay }"
            @click.stop="$emit('expand')"
            :disabled="shifts.length >= 3 || isPastDay"
            :title="getNewButtonTitle"
          >
            <span class="material-icons-round">add</span>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue';
import { format, isBefore, startOfDay } from 'date-fns';
import ShiftPill from './ShiftPill.vue';

interface ShiftTime {
  startTime: string;
  endTime: string;
  isHoliday?: boolean;
}

interface Shift extends ShiftTime {
  id: number;
}

const props = defineProps<{
  date: Date;
  shifts: Shift[];
  accentColor: string;
  clipboardShift: ShiftTime | null;
  isExpanded: boolean;
  isSelected: boolean;
}>();

const emit = defineEmits<{
  (e: 'update-shifts', shifts: Shift[]): void;
  (e: 'copy-shift', shift: ShiftTime): void;
  (e: 'clear-clipboard'): void;
  (e: 'expand'): void;
  (e: 'edit', shift: Shift): void;
}>();

const formattedWeekday = computed(() => format(props.date, 'EEE'));
const formattedDay = computed(() => format(props.date, 'd'));

const sortedShifts = computed(() => {
  return [...props.shifts].sort((a, b) => a.startTime.localeCompare(b.startTime));
});

const isPastDay = computed(() => {
  const today = startOfDay(new Date());
  return isBefore(props.date, today);
});

const timeToMinutes = (time: string): number => {
  const [hours, minutes] = time.split(':').map(Number);
  return hours * 60 + minutes;
};

const doShiftsOverlap = (shift1: ShiftTime, shift2: ShiftTime): boolean => {
  const start1 = timeToMinutes(shift1.startTime);
  const end1 = timeToMinutes(shift1.endTime);
  const start2 = timeToMinutes(shift2.startTime);
  const end2 = timeToMinutes(shift2.endTime);

  const adjustedEnd1 = end1 < start1 ? end1 + 24 * 60 : end1;
  const adjustedEnd2 = end2 < start2 ? end2 + 24 * 60 : end2;

  return (start1 < adjustedEnd2 && adjustedEnd1 > start2) ||
         (start2 < adjustedEnd1 && adjustedEnd2 > start1);
};

const wouldClipboardShiftOverlap = computed(() => {
  if (!props.clipboardShift) return false;
  return props.shifts.some(shift => doShiftsOverlap(props.clipboardShift!, shift));
});

const hasClipboardShift = computed(() => {
  if (!props.clipboardShift) return false;
  return props.shifts.some(shift => 
    shift.startTime === props.clipboardShift!.startTime && 
    shift.endTime === props.clipboardShift!.endTime
  );
});

const getNewButtonTitle = computed(() => {
  if (isPastDay.value) return 'Cannot add shifts to past days';
  if (props.shifts.length >= 3) return 'Maximum shifts reached';
  return 'Add new shift';
});

const getAddButtonTitle = computed(() => {
  if (isPastDay.value) return 'Cannot add shifts to past days';
  if (props.shifts.length >= 3) return 'Maximum shifts reached';
  if (hasClipboardShift.value) return 'This shift has already been added';
  if (wouldClipboardShiftOverlap.value) return 'This shift would overlap with an existing shift';
  return 'Add copied shift';
});

const quickDelete = (shiftId: number) => {
  if (isPastDay.value) return;
  emit('update-shifts', props.shifts.filter(shift => shift.id !== shiftId));
};

const applyClipboardShift = () => {
  if (!props.clipboardShift || 
      props.shifts.length >= 3 || 
      hasClipboardShift.value || 
      wouldClipboardShiftOverlap.value ||
      isPastDay.value) return;
  
  const newShift: Shift = {
    id: Date.now(),
    startTime: props.clipboardShift.startTime,
    endTime: props.clipboardShift.endTime,
    isHoliday: props.clipboardShift.isHoliday
  };
  
  emit('update-shifts', [...props.shifts, newShift]);
};
</script>

<style lang="scss" scoped>
@use '../../styles/main' as *;

.day-row {
  border: 1px solid rgba($border, 0.6);
  border-radius: 12px;
  background: rgba(white, 0.8);
  backdrop-filter: blur(8px);
  overflow: hidden;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  box-shadow: $shadow-sm;

  &.is-selected {
    border-color: $primary;
    background: rgba($primary, 0.02);
  }

  &.is-past {
    opacity: 0.75;
    background: rgba($text, 0.02);
    border-color: rgba($text, 0.1);

    .day-row__header {
      cursor: default;

      &:hover {
        background-color: transparent;
      }
    }

    .day-row__weekday,
    .day-row__day {
      color: rgba($text, 0.6);
    }

    .day-row__empty {
      color: rgba($text, 0.4);

      .material-icons-round {
        opacity: 0.4;
      }
    }

    .day-row__action-btn {
      &.is-disabled {
        opacity: 0.3;
        background: rgba($text, 0.05) !important;
        color: rgba($text, 0.3) !important;
      }
    }
  }

  &__header {
    display: flex;
    align-items: center;
    padding: $spacing-lg;
    cursor: pointer;
    gap: $spacing-md;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    -webkit-tap-highlight-color: transparent;

    @media (max-width: $breakpoint-sm) {
      padding: $spacing-md;
      gap: $spacing-md;
    }

    &:hover {
      background-color: rgba($primary, 0.03);
    }

    &:active {
      background-color: rgba($primary, 0.06);
    }
  }

  &__date {
    display: flex;
    flex-direction: column;
    min-width: 64px;
    flex-shrink: 0;

    @media (max-width: $breakpoint-sm) {
      min-width: 30px;
    }
  }

  &__weekday {
    font-size: $font-size-sm;
    color: rgba($text-light, 0.9);
    font-weight: $font-weight-medium;
    transition: color 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    letter-spacing: 0.01em;
  }

  &__day {
    font-size: 1.5rem;
    font-weight: $font-weight-bold;
    color: $text;
    line-height: 1.1;
    transition: color 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    letter-spacing: -0.02em;

    @media (max-width: $breakpoint-sm) {
      font-size: 1.25rem;
    }
  }

  &__shifts {
    flex: 1;
    display: flex;
    gap: $spacing-sm;
    align-items: center;
    min-width: 0;
    overflow: hidden;

    @media (max-width: $breakpoint-sm) {
      gap: $spacing-sm;
    }
  }

  &__actions {
    display: flex;
    gap: $spacing-xs;
    margin-left: auto;
    flex-shrink: 0;
  }

  &__action-btn {
    padding: $spacing-xs $spacing-md;
    border-radius: 999px;
    border: none;
    font-size: $font-size-sm;
    cursor: pointer;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    display: inline-flex;
    align-items: center;
    gap: $spacing-xs;
    font-weight: $font-weight-medium;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
    -webkit-tap-highlight-color: transparent;

    @media (max-width: $breakpoint-sm) {
      padding: $spacing-xs;
    }

    .material-icons-round {
      font-size: 1.2rem;
    }

    &.is-disabled {
      opacity: 0.4;
      cursor: not-allowed;
      background: rgba($text, 0.08) !important;
      color: rgba($text, 0.3) !important;
      box-shadow: none;
    }

    &:active:not(.is-disabled) {
      transform: translate3d(0, 0, 0) scale(0.96);
      box-shadow: none;
    }
  }

  &__new-btn,
  &__add-btn {
    width: 38px;
    height: 38px;
    padding: 0;
    justify-content: center;
  }

  &__new-btn {
    background: rgba($success, 0.15);
    color: $success;

    &:hover:not(.is-disabled) {
      background: rgba($success, 0.2);
      box-shadow: 0 2px 4px rgba($success, 0.2);
    }
  }

  &__add-btn {
    background: rgba($secondary, 0.15);
    color: $secondary;

    &:hover:not(.is-disabled) {
      background: rgba($secondary, 0.2);
      box-shadow: 0 2px 4px rgba($secondary, 0.2);
    }
  }

  &__empty {
    color: rgba($text-light, 0.8);
    font-size: $font-size-sm;
    display: flex;
    align-items: center;
    gap: $spacing-xs;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    letter-spacing: 0.01em;

    .material-icons-round {
      font-size: 1.1rem;
      opacity: 0.6;
      transition: opacity 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    }
  }
}
</style>
