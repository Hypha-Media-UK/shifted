<template>
  <div class="shift-editor">
    <div class="shift-editor__container">
      <div class="shift-editor__form">
        <div class="form-group">
          <label class="form-label">
            <span class="material-icons-round">schedule</span>
            Start Time
          </label>
          <input 
            type="time" 
            class="form-control" 
            v-model="props.shift.startTime"
            :disabled="props.disabled"
            :class="{ 'has-error': props.hasOverlap }"
          >
        </div>
        <div class="form-group">
          <label class="form-label">
            <span class="material-icons-round">schedule</span>
            End Time
          </label>
          <input 
            type="time" 
            class="form-control" 
            v-model="props.shift.endTime"
            :disabled="props.disabled"
            :class="{ 'has-error': props.hasOverlap }"
          >
        </div>
      </div>
      
      <div class="shift-editor__actions">
        <template v-if="props.isEditing">
          <div class="shift-editor__action-group">
            <button class="btn btn-secondary btn-icon" @click="$emit('cancel')">
              <span class="material-icons-round">close</span>
            </button>
            <div class="shift-editor__action-pair">
              <button class="btn btn-danger btn-icon" @click="$emit('delete')">
                <span class="material-icons-round">delete</span>
              </button>
              <button 
                class="btn btn-success btn-icon" 
                @click="$emit('update')"
                :disabled="!isValid || props.hasOverlap"
              >
                <span class="material-icons-round">check</span>
              </button>
              <button 
                class="btn btn-primary btn-icon" 
                @click="$emit('apply-and-copy')"
                :disabled="!isValid || props.hasOverlap"
              >
                <span class="material-icons-round">content_copy</span>
              </button>
            </div>
          </div>
        </template>
        <template v-else>
          <div class="shift-editor__action-group">
            <button class="btn btn-secondary btn-icon" @click="$emit('cancel')">
              <span class="material-icons-round">close</span>
            </button>
            <div class="shift-editor__action-pair">
              <button 
                class="btn btn-holiday btn-icon" 
                :class="{ 'is-active': props.shift.isHoliday }"
                @click="applyAsHoliday"
                :disabled="props.disabled || !isValid || props.hasOverlap"
              >
                <span class="material-icons-round">beach_access</span>
              </button>
              <button 
                class="btn btn-success btn-icon" 
                @click="$emit('apply')"
                :disabled="!isValid || props.hasOverlap || props.disabled"
              >
                <span class="material-icons-round">check</span>
              </button>
            </div>
          </div>
        </template>
      </div>
    </div>

    <Transition name="fade">
      <div v-if="props.warning" class="shift-editor__warning">
        <span class="material-icons-round">warning</span>
        {{ props.warning }}
      </div>
    </Transition>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue';

interface ShiftTime {
  startTime: string;
  endTime: string;
  isHoliday?: boolean;
}

const props = defineProps<{
  shift: ShiftTime;
  isEditing: boolean;
  disabled: boolean;
  hasOverlap: boolean;
  warning: string;
}>();

const emit = defineEmits<{
  (e: 'update'): void;
  (e: 'delete'): void;
  (e: 'cancel'): void;
  (e: 'apply'): void;
  (e: 'apply-and-copy'): void;
  (e: 'toggle-holiday'): void;
}>();

const isValid = computed(() => {
  if (!props.shift.startTime || !props.shift.endTime) return false;
  
  const [startHours, startMinutes] = props.shift.startTime.split(':').map(Number);
  const [endHours, endMinutes] = props.shift.endTime.split(':').map(Number);
  
  const startTotalMinutes = startHours * 60 + startMinutes;
  const endTotalMinutes = endHours * 60 + endMinutes;
  
  const adjustedEndMinutes = endTotalMinutes < startTotalMinutes 
    ? endTotalMinutes + (24 * 60) 
    : endTotalMinutes;
  
  return adjustedEndMinutes - startTotalMinutes <= 24 * 60;
});

const applyAsHoliday = () => {
  if (!isValid.value || props.hasOverlap || props.disabled) return;
  props.shift.isHoliday = true;
  emit('apply');
};
</script>

<style lang="scss" scoped>
@import '../../styles/abstracts/variables';

.shift-editor {
  background-color: rgba($background, 0.5);

  &__container {
    display: flex;
    gap: $spacing-lg;
    align-items: flex-end;
    padding: $spacing-md $spacing-lg;

    @media (max-width: $breakpoint-sm) {
      flex-direction: column;
      gap: $spacing-md;
      padding: $spacing-sm;
    }
  }

  &__form {
    display: flex;
    gap: $spacing-md;
    flex: 1;
    min-width: 0;
    align-items: flex-end;

    @media (max-width: $breakpoint-sm) {
      flex-direction: column;
      width: 100%;
      align-items: stretch;
    }

    .form-group {
      flex: 1;
      min-width: 0;
    }
  }

  &__warning {
    margin-top: 0;
    padding: $spacing-sm $spacing-md;
    background-color: rgba($danger, 0.1);
    color: $danger;
    font-size: $font-size-sm;
    display: flex;
    align-items: center;
    gap: $spacing-xs;
    border-top: 1px solid rgba($danger, 0.1);

    .material-icons-round {
      font-size: 1.1rem;
    }
  }

  &__actions {
    flex: 0 0 auto;

    @media (max-width: $breakpoint-sm) {
      width: 100%;
    }
  }

  &__action-group {
    display: flex;
    gap: $spacing-sm;
    align-items: center;

    @media (max-width: $breakpoint-sm) {
      flex-direction: column;
      width: 100%;

      .btn {
        width: 100%;
        justify-content: center;
      }

      .shift-editor__action-pair {
        display: flex;
        gap: $spacing-xs;
        width: 100%;
        justify-content: flex-end;

        .btn {
          width: auto;
        }
      }
    }
  }

  &__action-pair {
    display: flex;
    gap: $spacing-xs;
    margin-left: $spacing-xs;

    .btn-icon {
      width: 32px;
      height: 32px;
      padding: 0;
      display: flex;
      align-items: center;
      justify-content: center;

      &.btn-wide {
        width: 64px;
      }
    }
  }
}

.form-group {
  margin: 0;
}

.form-label {
  display: flex;
  align-items: center;
  gap: $spacing-xs;
  margin-bottom: $spacing-xs;
  color: $text;
  font-weight: $font-weight-medium;
  font-size: $font-size-sm;

  .material-icons-round {
    font-size: 1.1rem;
    color: $text-light;
  }
}

.form-control {
  width: 100%;
  padding: $spacing-xs $spacing-sm;
  border: 2px solid $border;
  border-radius: $border-radius-sm;
  font-size: $font-size-base;
  font-family: $font-family-base;
  transition: all $transition-speed ease;
  background-color: white;
  color: $text;
  appearance: none;
  height: 32px;
  
  &:hover:not(:disabled) {
    border-color: $text-light;
  }
  
  &:focus:not(:disabled) {
    outline: none;
    border-color: $primary;
    box-shadow: 0 0 0 3px rgba($primary, 0.1);
  }

  &:disabled {
    background-color: rgba($text, 0.05);
    color: $text-light;
    cursor: not-allowed;
  }

  &.has-error {
    border-color: $danger;
    
    &:focus {
      box-shadow: 0 0 0 3px rgba($danger, 0.1);
    }
  }
}

.btn {
  &.btn-icon {
    width: 32px;
    height: 32px;
    padding: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: $font-size-sm;
    font-weight: $font-weight-medium;

    .material-icons-round {
      font-size: 1.1rem;
    }

    &.btn-wide {
      width: 64px;
    }
  }
}

.btn-success {
  background: rgba($success, 0.25);
  color: $success;
  border: none;

  &:hover:not(:disabled) {
    background: rgba($success, 0.3);
    transform: translateY(-1px);
  }

  &:active:not(:disabled) {
    transform: translateY(0);
  }

  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
}

.btn-primary {
  background: rgba($primary, 0.25);
  color: $primary;
  border: none;

  &:hover:not(:disabled) {
    background: rgba($primary, 0.3);
    transform: translateY(-1px);
  }

  &:active:not(:disabled) {
    transform: translateY(0);
  }

  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
}

.btn-holiday {
  background: rgba($holiday, 0.25);
  color: $holiday;
  border: none;

  &:hover:not(:disabled) {
    background: rgba($holiday, 0.3);
    transform: translateY(-1px);
  }

  &:active:not(:disabled),
  &.is-active:not(:disabled) {
    background: rgba($holiday, 0.4);
    color: darken($holiday, 5%);
    transform: translateY(0);
  }

  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
}

// Transitions
.fade-enter-active,
.fade-leave-active {
  transition: all $transition-speed ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translateY(-$spacing-sm);
}
</style>
