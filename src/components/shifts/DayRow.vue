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
            @edit="editShift(shift)"
            @delete="quickDelete(shift.id)"
          />
        </template>
        <div v-else class="day-row__empty" :class="{ 'is-past': isPastDay }">
          <span class="material-icons-round">schedule</span>
          No shifts
        </div>
        <div class="day-row__actions">
          <button 
            class="day-row__action-btn day-row__new-btn"
            :class="{ 'is-disabled': shifts.length >= 3 || isPastDay }"
            @click.stop="handleNewClick"
            :disabled="shifts.length >= 3 || isPastDay"
            :title="getNewButtonTitle"
          >
            <span class="material-icons-round">add</span>
          </button>
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
        </div>
      </div>
    </div>

    <Transition name="slide-up">
      <div v-if="isExpanded" class="day-row__content">
        <ShiftEditor
          :shift="currentShift"
          :is-editing="isEditing"
          :disabled="shifts.length >= 3 && !isEditing || isPastDay"
          :has-overlap="hasOverlap"
          :warning="validationWarning"
          @update="updateShift"
          @delete="deleteShift"
          @cancel="cancelEdit"
          @apply="applyShift"
          @apply-and-copy="copyShift"
          @toggle-holiday="toggleHoliday"
        />
      </div>
    </Transition>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
import { format, isBefore, startOfDay } from 'date-fns';
import ShiftEditor from './ShiftEditor.vue';
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
}>();

const isEditing = ref(false);
const editingShiftId = ref<number | null>(null);
const currentShift = ref<ShiftTime>({
  startTime: '',
  endTime: '',
  isHoliday: false
});

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

const hasOverlap = computed(() => {
  if (!currentShift.value.startTime || !currentShift.value.endTime) return false;
  
  return props.shifts.some(shift => {
    if (isEditing.value && shift.id === editingShiftId.value) return false;
    return doShiftsOverlap(currentShift.value, shift);
  });
});

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

const validationWarning = computed(() => {
  if (!currentShift.value.startTime || !currentShift.value.endTime) return '';
  if (hasOverlap.value) return 'This shift overlaps with an existing shift';
  if (isPastDay.value) return 'Cannot modify shifts in past days';
  return '';
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

const handleNewClick = () => {
  if (props.shifts.length >= 3 || isPastDay.value) return;
  emit('expand');
};

const quickDelete = (shiftId: number) => {
  if (isPastDay.value) return;
  emit('update-shifts', props.shifts.filter(shift => shift.id !== shiftId));
};

const applyShift = () => {
  if (hasOverlap.value || isPastDay.value) return;
  
  const newShift: Shift = {
    id: Date.now(),
    startTime: currentShift.value.startTime,
    endTime: currentShift.value.endTime,
    isHoliday: currentShift.value.isHoliday
  };
  
  emit('update-shifts', [...props.shifts, newShift]);
  emit('clear-clipboard');
  resetForm();
  emit('expand');
};

const copyShift = () => {
  if (hasOverlap.value || isPastDay.value) return;
  emit('copy-shift', { ...currentShift.value });
  resetForm();
  emit('expand');
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

const updateShift = () => {
  if (hasOverlap.value || isPastDay.value) return;
  
  const updatedShifts = props.shifts.map(shift => 
    shift.id === editingShiftId.value 
      ? { 
          ...shift, 
          startTime: currentShift.value.startTime,
          endTime: currentShift.value.endTime,
          isHoliday: currentShift.value.isHoliday
        } 
      : shift
  );
  
  emit('update-shifts', updatedShifts);
  resetForm();
  emit('expand');
};

const deleteShift = () => {
  if (isPastDay.value) return;
  emit('update-shifts', props.shifts.filter(shift => shift.id !== editingShiftId.value));
  resetForm();
  emit('expand');
};

const editShift = (shift: Shift) => {
  if (isPastDay.value) return;
  isEditing.value = true;
  editingShiftId.value = shift.id;
  currentShift.value = { 
    startTime: shift.startTime,
    endTime: shift.endTime,
    isHoliday: shift.isHoliday
  };
  emit('clear-clipboard');
  emit('expand');
};

const cancelEdit = () => {
  resetForm();
  emit('expand');
};

const toggleHoliday = () => {
  currentShift.value.isHoliday = !currentShift.value.isHoliday;
};

const resetForm = () => {
  currentShift.value = { startTime: '', endTime: '', isHoliday: false };
  isEditing.value = false;
  editingShiftId.value = null;
};
</script>

<style lang="scss" scoped>
@import '../../styles/abstracts/variables';

.day-row {
  border: 1px solid $border;
  border-radius: $border-radius-lg;
  background: white;
  overflow: hidden;
  transition: all $transition-speed ease;

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
    padding: $spacing-md $spacing-lg;
    cursor: pointer;
    gap: $spacing-md;
    transition: background-color 0.15s ease;

    @media (max-width: $breakpoint-sm) {
      padding: $spacing-sm;
      gap: $spacing-sm;
    }

    &:hover {
      background-color: rgba($primary, 0.05);
    }
  }

  &__date {
    display: flex;
    flex-direction: column;
    min-width: 60px;
    flex-shrink: 0;

    @media (max-width: $breakpoint-sm) {
      min-width: 45px;
    }
  }

  &__weekday {
    font-size: $font-size-sm;
    color: $text-light;
    font-weight: $font-weight-medium;
    transition: color $transition-speed ease;
  }

  &__day {
    font-size: $font-size-xl;
    font-weight: $font-weight-bold;
    color: $text;
    line-height: 1.2;
    transition: color $transition-speed ease;

    @media (max-width: $breakpoint-sm) {
      font-size: $font-size-lg;
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
      gap: $spacing-xs;
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
    border-radius: $border-radius;
    border: none;
    font-size: $font-size-sm;
    cursor: pointer;
    transition: all $transition-speed ease;
    display: inline-flex;
    align-items: center;
    gap: $spacing-xs;
    font-weight: $font-weight-medium;

    @media (max-width: $breakpoint-sm) {
      padding: $spacing-xs;
    }

    .material-icons-round {
      font-size: 1.2rem;
    }

    &.is-disabled {
      opacity: 0.5;
      cursor: not-allowed;
      background: rgba($text, 0.1) !important;
      color: rgba($text, 0.4) !important;
    }
  }

  &__new-btn {
    background: rgba($success, 0.25);
    color: $success;
    width: 32px;
    height: 32px;
    padding: 0;
    justify-content: center;

    &:hover:not(.is-disabled) {
      background: rgba($success, 0.3);
      transform: translateY(-1px);
    }

    &:active:not(.is-disabled) {
      transform: translateY(0);
    }
  }

  &__add-btn {
    background: rgba($secondary, 0.1);
    color: $secondary;

    &:hover:not(.is-disabled) {
      background: rgba($secondary, 0.15);
    }
  }

  &__empty {
    color: $text-light;
    font-size: $font-size-sm;
    display: flex;
    align-items: center;
    gap: $spacing-xs;
    transition: all $transition-speed ease;

    .material-icons-round {
      font-size: 1.1rem;
      opacity: 0.7;
      transition: opacity $transition-speed ease;
    }
  }

  &__content {
    border-top: 1px solid $border;
  }
}

// Transitions
.slide-up-enter-active,
.slide-up-leave-active {
  transition: all $transition-speed $transition-bounce;
  max-height: 400px;
}

.slide-up-enter-from,
.slide-leave-to {
  max-height: 0;
  opacity: 0;
}

.hide-sm {
  @media (max-width: $breakpoint-sm) {
    display: none;
  }
}
</style>
