<template>
  <div class="shift-editor" @click.self="$emit('cancel')">
    <div class="shift-editor__overlay" @click="$emit('cancel')"></div>
    <div class="shift-editor__content">
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
              v-model="shift.startTime"
              :disabled="disabled"
              :class="{ 'has-error': hasOverlap }"
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
              v-model="shift.endTime"
              :disabled="disabled"
              :class="{ 'has-error': hasOverlap }"
            >
          </div>
        </div>
      
        <div class="shift-editor__actions">
          <template v-if="isEditing">
            <div class="shift-editor__action-group">
              <div class="shift-editor__action-pair">
                <button class="btn btn-danger btn-icon" @click="$emit('delete')">
                  <span class="material-icons-round">delete</span>
                </button>
                <button 
                  v-if="!shift.isHoliday"
                  class="btn btn-holiday btn-icon" 
                  @click="toggleHoliday"
                  title="Mark as holiday"
                >
                  <span class="material-icons-round">beach_access</span>
                </button>
                <button 
                  v-else
                  class="btn btn-revert btn-icon" 
                  @click="toggleHoliday"
                  title="Change back to standard shift"
                >
                  <span class="material-icons-round">settings_backup_restore</span>
                </button>
                <button 
                  class="btn btn-primary btn-icon" 
                  @click="$emit('apply-and-copy')"
                  :disabled="!isValid || hasOverlap"
                >
                  <span class="material-icons-round">content_copy</span>
                </button>
                <button 
                  class="btn btn-success btn-icon" 
                  @click="$emit('update')"
                  :disabled="!isValid || hasOverlap"
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
                  v-if="!shift.isHoliday"
                  class="btn btn-holiday btn-icon" 
                  @click="toggleHoliday"
                  title="Mark as holiday"
                >
                  <span class="material-icons-round">beach_access</span>
                </button>
                <button 
                  v-else
                  class="btn btn-revert btn-icon" 
                  @click="toggleHoliday"
                  title="Change back to standard shift"
                >
                  <span class="material-icons-round">settings_backup_restore</span>
                </button>
                <button 
                  class="btn btn-success btn-icon" 
                  @click="$emit('apply')"
                  :disabled="!isValid || hasOverlap || disabled"
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
        <div v-if="warning" class="shift-editor__warning">
          <span class="material-icons-round">warning</span>
          {{ warning }}
        </div>
      </Transition>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, toRefs, watch } from 'vue';

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

const { shift, isEditing, disabled, hasOverlap, warning } = toRefs(props);

// Watch for startTime changes and set endTime one hour ahead for new shifts
watch(() => shift.value.startTime, (newStartTime) => {
  if (!isEditing.value && newStartTime && !shift.value.endTime) {
    const [hours, minutes] = newStartTime.split(':').map(Number);
    const endHours = (hours + 1) % 24;
    shift.value.endTime = `${endHours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}`;
  }
});

const emit = defineEmits<{
  (e: 'update'): void;
  (e: 'delete'): void;
  (e: 'cancel'): void;
  (e: 'apply'): void;
  (e: 'apply-and-copy'): void;
  (e: 'toggle-holiday', shift: ShiftTime): void;
}>();

const isValid = computed(() => {
  if (!shift.value.startTime || !shift.value.endTime) return false;
  
  const [startHours, startMinutes] = shift.value.startTime.split(':').map(Number);
  const [endHours, endMinutes] = shift.value.endTime.split(':').map(Number);
  
  const startTotalMinutes = startHours * 60 + startMinutes;
  const endTotalMinutes = endHours * 60 + endMinutes;
  
  const adjustedEndMinutes = endTotalMinutes < startTotalMinutes 
    ? endTotalMinutes + (24 * 60) 
    : endTotalMinutes;
  
  return adjustedEndMinutes - startTotalMinutes <= 24 * 60;
});

const toggleHoliday = () => {
  shift.value.isHoliday = !shift.value.isHoliday;
  if (isEditing.value) {
    emit('toggle-holiday', shift.value);
  }
};
</script>

<style lang="scss" scoped>
@use '../../styles/main' as *;

.shift-editor {
  position: fixed;
  inset: 0;
  z-index: 2000;
  display: grid;
  place-items: center;
  padding: $spacing-lg;

  @media (max-width: $breakpoint-sm) {
    padding: $spacing-md;
  }

  &__overlay {
    position: absolute;
    inset: 0;
    background-color: rgba(black, 0.4);
    backdrop-filter: blur(12px);
    animation: fade-in 0.3s ease-out;
    will-change: opacity;
  }

  &__content {
    position: relative;
    z-index: 2001;
    width: 100%;
    max-width: 480px;
    background-color: white;
    border-radius: 24px;
    box-shadow: 0 8px 32px rgba(black, 0.12);
    animation: scale-in 0.4s cubic-bezier(0.34, 1.56, 0.64, 1);
    transform-origin: center;
    will-change: transform, opacity;
    overflow: hidden;
  }

  &__container {
    display: grid;
    gap: $spacing-lg;
    padding: $spacing-lg;

    @media (max-width: $breakpoint-sm) {
      gap: $spacing-md;
      padding: $spacing-md;
    }
  }

  &__form {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: $spacing-md;
    min-width: 0;

    @media (max-width: $breakpoint-sm) {
      grid-template-columns: 1fr;
    }

    .form-group {
      min-width: 0;
    }
  }

  &__warning {
    position: relative;
    z-index: 2001;
    padding: $spacing-md;
    background-color: white;
    color: $danger;
    font-size: $font-size-sm;
    display: flex;
    align-items: center;
    gap: $spacing-xs;
    border-top: 1px solid rgba($danger, 0.1);
    letter-spacing: 0.01em;

    .material-icons-round {
      font-size: 1.1rem;
    }
  }

  &__actions {
    display: flex;
    justify-content: flex-end;
    min-width: 0;
  }

  &__action-group {
    display: flex;
    gap: $spacing-sm;
    align-items: center;
    justify-content: flex-end;
  }

  &__action-pair {
    display: flex;
    gap: $spacing-sm;
    align-items: center;
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
  border-radius: 12px;
  font-size: $font-size-base;
  font-family: $font-family-base;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  background-color: white;
  color: $text;
  appearance: none;
  height: 46px;
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

    &:active:not(:disabled) {
      transform: translate3d(0, 0, 0) scale(0.96);
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

  &:disabled {
    opacity: 0.4;
    cursor: not-allowed;
    background: rgba($text, 0.08) !important;
    color: rgba($text, 0.3) !important;
    box-shadow: none;
  }
}

.btn-revert {
  background: rgba($text, 0.1);
  color: rgba($text, 0.7);

  &:hover:not(:disabled) {
    background: rgba($text, 0.15);
    box-shadow: 0 2px 4px rgba($text, 0.15);
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

@keyframes fade-in {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

@keyframes scale-in {
  0% {
    opacity: 0;
    transform: translate3d(0, 0, 0) scale(0.95);
  }
  60% {
    transform: translate3d(0, 0, 0) scale(1.02);
  }
  100% {
    opacity: 1;
    transform: translate3d(0, 0, 0) scale(1);
  }
}
</style>
