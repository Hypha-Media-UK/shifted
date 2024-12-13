<template>
  <div class="day-row" :class="{ 'is-expanded': isExpanded }">
    <div class="day-row__header">
      <div class="day-row__date">
        <span class="day-row__weekday">{{ formattedWeekday }}</span>
        <span class="day-row__day">{{ formattedDay }}</span>
      </div>
      <div class="day-row__shifts">
        <template v-if="shifts.length">
          <div 
            v-for="shift in sortedShifts" 
            :key="shift.id" 
            class="day-row__shift-pill"
            :style="{ backgroundColor: $props.accentColor + '33' }"
          >
            <span @click.stop="editShift(shift)">
              {{ formatShiftTime(shift.startTime) }} - {{ formatShiftTime(shift.endTime) }}
            </span>
            <button 
              class="day-row__quick-delete" 
              @click.stop="quickDelete(shift.id)"
              title="Quick delete"
            >
              ×
            </button>
          </div>
        </template>
        <div v-else class="day-row__empty">
          No shifts
        </div>
        <div class="day-row__actions">
          <button 
            class="day-row__action-btn day-row__new-btn"
            :class="{ 'is-disabled': shifts.length >= 3 }"
            @click.stop="handleNewClick"
            :disabled="shifts.length >= 3"
            :title="shifts.length >= 3 ? 'Maximum shifts reached' : 'Add new shift'"
          >
            New
          </button>
          <button 
            v-if="clipboardShift"
            class="day-row__action-btn day-row__add-btn"
            :class="{ 'is-disabled': shifts.length >= 3 || hasClipboardShift || wouldClipboardShiftOverlap }"
            @click.stop="applyClipboardShift"
            :disabled="shifts.length >= 3 || hasClipboardShift || wouldClipboardShiftOverlap"
            :title="getAddButtonTitle"
          >
            Add
          </button>
        </div>
      </div>
      <div class="day-row__expand" v-if="isExpanded">
        <span class="icon" :class="{ 'is-rotated': isExpanded }">▼</span>
      </div>
    </div>

    <Transition name="slide-down">
      <div v-if="isExpanded" class="day-row__content">
        <div class="shift-editor">
          <div class="form-group">
            <label>Start Time</label>
            <input 
              type="time" 
              class="form-control" 
              v-model="currentShift.startTime"
              :disabled="shifts.length >= 3 && !isEditing"
            >
          </div>
          <div class="form-group">
            <label>End Time</label>
            <input 
              type="time" 
              class="form-control" 
              v-model="currentShift.endTime"
              :disabled="shifts.length >= 3 && !isEditing"
            >
          </div>
          
          <div v-if="validationWarning" class="shift-editor__warning">
            {{ validationWarning }}
          </div>

          <div class="shift-editor__actions">
            <template v-if="isEditing">
              <button 
                class="btn btn-primary" 
                @click="updateShift"
                :disabled="!isValidShift || hasOverlap"
              >
                Update
              </button>
              <button class="btn btn-danger" @click="deleteShift">Delete</button>
              <button class="btn btn-secondary" @click="cancelEdit">Cancel</button>
            </template>
            <template v-else>
              <button 
                class="btn btn-primary" 
                @click="applyShift"
                :disabled="!isValidShift || hasOverlap || shifts.length >= 3"
              >
                Apply Once
              </button>
              <button 
                class="btn btn-secondary" 
                @click="applyAndAddMore"
                :disabled="!isValidShift || hasOverlap || shifts.length >= 3"
              >
                Apply and Add to More
              </button>
              <button class="btn btn-secondary" @click="cancel">Cancel</button>
            </template>
          </div>
        </div>
      </div>
    </Transition>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue';
import { format } from 'date-fns';

interface ShiftTime {
  startTime: string;
  endTime: string;
}

interface Shift extends ShiftTime {
  id: number;
}

const props = defineProps({
  date: {
    type: Date,
    required: true
  },
  shifts: {
    type: Array as () => Shift[],
    default: () => []
  },
  accentColor: {
    type: String,
    default: '#4A90E2'
  },
  clipboardShift: {
    type: Object as () => ShiftTime | null,
    default: null
  },
  isExpanded: {
    type: Boolean,
    default: false
  }
});

const emit = defineEmits(['update-shifts', 'copy-shift', 'clear-clipboard', 'expand']);

const isEditing = ref(false);
const editingShiftId = ref<number | null>(null);
const currentShift = ref<ShiftTime>({
  startTime: '',
  endTime: ''
});

watch(() => props.isExpanded, (newValue) => {
  if (!newValue) {
    resetForm();
  }
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

const formattedWeekday = computed(() => format(props.date, 'EEE'));
const formattedDay = computed(() => format(props.date, 'd'));

const sortedShifts = computed(() => {
  return [...props.shifts].sort((a, b) => {
    return a.startTime.localeCompare(b.startTime);
  });
});

const isValidShift = computed(() => {
  if (!currentShift.value.startTime || !currentShift.value.endTime) return false;
  
  const startTotalMinutes = timeToMinutes(currentShift.value.startTime);
  const endTotalMinutes = timeToMinutes(currentShift.value.endTime);
  
  const adjustedEndMinutes = endTotalMinutes < startTotalMinutes 
    ? endTotalMinutes + (24 * 60) 
    : endTotalMinutes;
  
  return adjustedEndMinutes - startTotalMinutes <= 24 * 60;
});

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
  
  if (!isValidShift.value) {
    return 'Shift duration cannot exceed 24 hours';
  }
  
  if (hasOverlap.value) {
    return 'This shift overlaps with an existing shift';
  }
  
  return '';
});

const getAddButtonTitle = computed(() => {
  if (props.shifts.length >= 3) return 'Maximum shifts reached';
  if (hasClipboardShift.value) return 'This shift has already been added';
  if (wouldClipboardShiftOverlap.value) return 'This shift would overlap with an existing shift';
  return 'Add copied shift';
});

const handleNewClick = () => {
  if (props.shifts.length >= 3) return;
  emit('expand');
};

const formatShiftTime = (time: string) => {
  const [hours, minutes] = time.split(':');
  return `${hours}:${minutes}`;
};

const quickDelete = (shiftId: number) => {
  const updatedShifts = props.shifts.filter(shift => shift.id !== shiftId);
  emit('update-shifts', updatedShifts);
};

const applyShift = () => {
  if (!isValidShift.value || hasOverlap.value) return;
  
  const newShift: Shift = {
    id: Date.now(),
    startTime: currentShift.value.startTime,
    endTime: currentShift.value.endTime
  };
  
  emit('update-shifts', [...props.shifts, newShift]);
  resetForm();
  emit('expand');
};

const applyAndAddMore = () => {
  if (!isValidShift.value || hasOverlap.value) return;
  
  const newShift: Shift = {
    id: Date.now(),
    startTime: currentShift.value.startTime,
    endTime: currentShift.value.endTime
  };
  
  emit('update-shifts', [...props.shifts, newShift]);
  emit('copy-shift', { 
    startTime: currentShift.value.startTime, 
    endTime: currentShift.value.endTime 
  });
  resetForm();
  emit('expand');
};

const applyClipboardShift = () => {
  if (!props.clipboardShift || 
      props.shifts.length >= 3 || 
      hasClipboardShift.value || 
      wouldClipboardShiftOverlap.value) return;
  
  const newShift: Shift = {
    id: Date.now(),
    startTime: props.clipboardShift.startTime,
    endTime: props.clipboardShift.endTime
  };
  
  emit('update-shifts', [...props.shifts, newShift]);
};

const updateShift = () => {
  if (!isValidShift.value || hasOverlap.value) return;
  
  const updatedShifts = props.shifts.map(shift => 
    shift.id === editingShiftId.value 
      ? { 
          ...shift, 
          startTime: currentShift.value.startTime,
          endTime: currentShift.value.endTime
        } 
      : shift
  );
  
  emit('update-shifts', updatedShifts);
  resetForm();
  emit('expand');
};

const deleteShift = () => {
  const updatedShifts = props.shifts.filter(shift => shift.id !== editingShiftId.value);
  emit('update-shifts', updatedShifts);
  resetForm();
  emit('expand');
};

const editShift = (shift: Shift) => {
  isEditing.value = true;
  editingShiftId.value = shift.id;
  currentShift.value = { 
    startTime: shift.startTime,
    endTime: shift.endTime
  };
  emit('clear-clipboard');
  emit('expand');
};

const cancelEdit = () => {
  resetForm();
  emit('expand');
};

const cancel = () => {
  resetForm();
  emit('expand');
};

const resetForm = () => {
  currentShift.value = { startTime: '', endTime: '' };
  isEditing.value = false;
  editingShiftId.value = null;
};
</script>

<style lang="scss" scoped>
@import '@/styles/abstracts/variables';

.day-row {
  border: 1px solid $border;
  border-radius: $border-radius;
  margin-bottom: $spacing-sm;
  background: white;
  overflow: hidden;

  &__header {
    display: flex;
    align-items: center;
    padding: $spacing-md;
    transition: background-color $transition-speed ease;
  }

  &__date {
    display: flex;
    flex-direction: column;
    min-width: 60px;
  }

  &__weekday {
    font-size: $font-size-sm;
    color: rgba($text, 0.6);
  }

  &__day {
    font-size: $font-size-lg;
    font-weight: 500;
  }

  &__shifts {
    flex: 1;
    display: flex;
    gap: $spacing-sm;
    margin: 0 $spacing-md;
    align-items: center;
  }

  &__shift-pill {
    padding: $spacing-xs $spacing-sm;
    border-radius: 16px;
    font-size: $font-size-sm;
    display: flex;
    align-items: center;
    gap: $spacing-xs;

    > span {
      cursor: pointer;
      transition: transform $transition-speed ease;

      &:hover {
        transform: translateY(-1px);
      }
    }
  }

  &__quick-delete {
    width: 20px;
    height: 20px;
    border-radius: 50%;
    border: none;
    background: rgba($danger, 0.1);
    color: $danger;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    font-size: 16px;
    line-height: 1;
    padding: 0;
    transition: all $transition-speed ease;

    &:hover {
      background: $danger;
      color: white;
    }
  }

  &__actions {
    display: flex;
    gap: $spacing-xs;
    margin-left: auto;
  }

  &__action-btn {
    padding: $spacing-xs $spacing-sm;
    border-radius: $border-radius;
    border: none;
    font-size: $font-size-sm;
    cursor: pointer;
    transition: all $transition-speed ease;
    display: flex;
    align-items: center;
    justify-content: center;
    font-weight: 500;

    &.is-disabled {
      opacity: 0.5;
      cursor: not-allowed;
      background: rgba($text, 0.1) !important;
      color: rgba($text, 0.4) !important;
    }
  }

  &__new-btn {
    background: rgba($primary, 0.1);
    color: $primary;

    &:hover:not(.is-disabled) {
      background: $primary;
      color: white;
    }
  }

  &__add-btn {
    background: rgba($secondary, 0.1);
    color: $secondary;

    &:hover:not(.is-disabled) {
      background: $secondary;
      color: white;
    }
  }

  &__empty {
    color: rgba($text, 0.4);
    font-size: $font-size-sm;
  }

  &__expand {
    .icon {
      transition: transform $transition-speed ease;
      display: inline-block;
      
      &.is-rotated {
        transform: rotate(180deg);
      }
    }
  }

  &__content {
    padding: $spacing-md;
    border-top: 1px solid $border;
    background-color: rgba($background, 0.5);
  }
}

.shift-editor {
  &__warning {
    margin-top: $spacing-sm;
    padding: $spacing-xs $spacing-sm;
    background-color: rgba($danger, 0.1);
    color: $danger;
    border-radius: $border-radius;
    font-size: $font-size-sm;
  }

  &__actions {
    display: flex;
    gap: $spacing-sm;
    margin-top: $spacing-md;
  }
}
</style>
