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
            <div class="shift-editor__action-pair">
              <button class="btn btn-danger btn-icon" @click="$emit('delete')">
                <span class="material-icons-round">delete</span>
              </button>
              <button 
                class="btn btn-primary btn-icon" 
                @click="$emit('apply-and-copy')"
                :disabled="!isValid || props.hasOverlap"
              >
                <span class="material-icons-round">content_copy</span>
              </button>
              <button 
                class="btn btn-success btn-icon" 
                @click="$emit('update')"
                :disabled="!isValid || props.hasOverlap"
              >
                <span class="material-icons-round">check</span>
              </button>
              <button class="btn btn-secondary btn-icon" @click="$emit('cancel')">
                <span class="material-icons-round">close</span>
              </button>
            </div>
          </div>
        </template>
        <template v-else>
          <div class="shift-editor__action-group">
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
              <button class="btn btn-secondary btn-icon" @click="$emit('cancel')">
                <span class="material-icons-round">close</span>
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
  background-color: rgba(white, 0.5);
  backdrop-filter: blur(8px);

  &__container {
    display: flex;
    gap: $spacing-lg;
    align-items: flex-end;
    padding: $spacing-lg;

    @media (max-width: $breakpoint-sm) {
      flex-direction: column;
      gap: $spacing-md;
      padding: $spacing-md;
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
    padding: $spacing-md;
    background-color: rgba($danger, 0.08);
    color: $danger;
    font-size: $font-size-sm;
    display: flex;
    align-items: center;
    gap: $spacing-xs;
    border-top: 1px solid rgba($danger, 0.1);
    backdrop-filter: blur(8px);
    letter-spacing: 0.01em;

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
      width: 100%;
      justify-content: flex-end;
      gap: $spacing-sm;
    }
  }

  &__action-pair {
    display: flex;
    gap: $spacing-sm;
    width: 100%;
    justify-content: flex-end;

    .btn-icon {
      width: 42px;
      height: 42px;
      padding: 0;
      display: flex;
      align-items: center;
      justify-content: center;

      @media (max-width: $breakpoint-sm) {
        width: 46px;
        height: 46px;

        .material-icons-round {
          font-size: 1.3rem;
        }
      }

      &.btn-wide {
        width: 84px;
        
        @media (max-width: $breakpoint-sm) {
          width: 92px;
        }
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
  color: rgba($text, 0.8);
  font-weight: $font-weight-medium;
  font-size: $font-size-sm;
  letter-spacing: 0.01em;

  .material-icons-round {
    font-size: 1.1rem;
    color: rgba($text-light, 0.9);
  }
}

.form-control {
  width: 100%;
  padding: $spacing-sm $spacing-md;
  border: 1px solid rgba($border, 0.8);
  border-radius: 10px;
  font-size: $font-size-base;
  font-family: $font-family-base;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  background-color: rgba(white, 0.8);
  color: $text;
  appearance: none;
  height: 42px;
  letter-spacing: 0.01em;
  -webkit-tap-highlight-color: transparent;
  
  &:hover:not(:disabled) {
    border-color: rgba($text-light, 0.6);
    background-color: rgba(white, 0.9);
  }
  
  &:focus:not(:disabled) {
    outline: none;
    border-color: $primary;
    background-color: white;
    box-shadow: 0 2px 4px rgba($primary, 0.1);
  }

  &:disabled {
    background-color: rgba($text, 0.03);
    color: rgba($text-light, 0.8);
    cursor: not-allowed;
    border-color: rgba($border, 0.4);
  }

  &.has-error {
    border-color: $danger;
    
    &:focus {
      box-shadow: 0 2px 4px rgba($danger, 0.1);
    }
  }
}

.btn {
  &.btn-icon {
    width: 42px;
    height: 42px;
    padding: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: $font-size-sm;
    font-weight: $font-weight-medium;
    border-radius: 999px;
    border: none;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    -webkit-tap-highlight-color: transparent;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);

    @media (max-width: $breakpoint-sm) {
      width: 46px;
      height: 46px;
      
      .material-icons-round {
        font-size: 1.3rem;
      }
    }

    &.btn-wide {
      width: 84px;

      @media (max-width: $breakpoint-sm) {
        width: 92px;
      }
    }

    &:active:not(:disabled) {
      transform: scale(0.96);
      box-shadow: none;
    }
  }
}

.btn-success {
  background: rgba($success, 0.15);
  color: $success;

  &:hover:not(:disabled) {
    background: rgba($success, 0.2);
    box-shadow: 0 2px 4px rgba($success, 0.2);
  }

  &:disabled {
    opacity: 0.4;
    cursor: not-allowed;
    background: rgba($text, 0.08) !important;
    color: rgba($text, 0.3) !important;
    box-shadow: none;
  }
}

.btn-primary {
  background: rgba($primary, 0.15);
  color: $primary;

  &:hover:not(:disabled) {
    background: rgba($primary, 0.2);
    box-shadow: 0 2px 4px rgba($primary, 0.2);
  }

  &:disabled {
    opacity: 0.4;
    cursor: not-allowed;
    background: rgba($text, 0.08) !important;
    color: rgba($text, 0.3) !important;
    box-shadow: none;
  }
}

.btn-holiday {
  background: rgba($holiday, 0.15);
  color: $holiday;

  &:hover:not(:disabled) {
    background: rgba($holiday, 0.2);
    box-shadow: 0 2px 4px rgba($holiday, 0.2);
  }

  &:active:not(:disabled),
  &.is-active:not(:disabled) {
    background: rgba($holiday, 0.25);
    color: darken($holiday, 5%);
  }

  &:disabled {
    opacity: 0.4;
    cursor: not-allowed;
    background: rgba($text, 0.08) !important;
    color: rgba($text, 0.3) !important;
    box-shadow: none;
  }
}

.btn-secondary {
  background: rgba($text, 0.08);
  color: rgba($text, 0.7);

  &:hover:not(:disabled) {
    background: rgba($text, 0.12);
    box-shadow: 0 2px 4px rgba($text, 0.1);
  }

  &:disabled {
    opacity: 0.4;
    cursor: not-allowed;
    background: rgba($text, 0.08) !important;
    color: rgba($text, 0.3) !important;
    box-shadow: none;
  }
}

.btn-danger {
  background: rgba($danger, 0.15);
  color: $danger;

  &:hover:not(:disabled) {
    background: rgba($danger, 0.2);
    box-shadow: 0 2px 4px rgba($danger, 0.2);
  }

  &:disabled {
    opacity: 0.4;
    cursor: not-allowed;
    background: rgba($text, 0.08) !important;
    color: rgba($text, 0.3) !important;
    box-shadow: none;
  }
}

// Transitions
.fade-enter-active {
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

.fade-leave-active {
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translateY(-4px);
}
</style>
