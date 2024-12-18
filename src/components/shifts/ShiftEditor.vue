<template>
  <div class="shift-editor">
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
    
    <Transition name="fade">
      <div v-if="warning" class="shift-editor__warning">
        <span class="material-icons-round">warning</span>
        {{ warning }}
      </div>
    </Transition>

    <div class="shift-editor__actions">
      <template v-if="isEditing">
        <button 
          class="btn btn-primary" 
          @click="$emit('update')"
          :disabled="!isValid || hasOverlap"
        >
          <span class="material-icons-round">save</span>
          <span class="hide-sm">Update</span>
        </button>
        <button class="btn btn-danger" @click="$emit('delete')">
          <span class="material-icons-round">delete</span>
          <span class="hide-sm">Delete</span>
        </button>
        <button class="btn btn-secondary" @click="$emit('cancel')">
          <span class="material-icons-round">close</span>
          <span class="hide-sm">Cancel</span>
        </button>
      </template>
      <template v-else>
        <button 
          class="btn btn-primary" 
          @click="$emit('apply')"
          :disabled="!isValid || hasOverlap || disabled"
        >
          <span class="material-icons-round">check</span>
          <span class="hide-sm">Apply Once</span>
        </button>
        <button 
          class="btn btn-secondary" 
          @click="$emit('apply-and-copy')"
          :disabled="!isValid || hasOverlap || disabled"
        >
          <span class="material-icons-round">content_copy</span>
          <span class="hide-sm">Apply and Copy</span>
        </button>
        <button class="btn btn-secondary" @click="$emit('cancel')">
          <span class="material-icons-round">close</span>
          <span class="hide-sm">Cancel</span>
        </button>
      </template>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue';

interface ShiftTime {
  startTime: string;
  endTime: string;
}

const props = defineProps<{
  shift: ShiftTime;
  isEditing: boolean;
  disabled: boolean;
  hasOverlap: boolean;
  warning: string;
}>();

defineEmits<{
  (e: 'update'): void;
  (e: 'delete'): void;
  (e: 'cancel'): void;
  (e: 'apply'): void;
  (e: 'apply-and-copy'): void;
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
</script>

<style lang="scss" scoped>
@import '../../styles/abstracts/variables';

.shift-editor {
  &__form {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: $spacing-md;

    @media (max-width: $breakpoint-sm) {
      grid-template-columns: 1fr;
      gap: $spacing-sm;
    }
  }

  &__warning {
    margin-top: $spacing-md;
    padding: $spacing-sm $spacing-md;
    background-color: rgba($danger, 0.1);
    color: $danger;
    border-radius: $border-radius-lg;
    font-size: $font-size-sm;
    display: flex;
    align-items: center;
    gap: $spacing-xs;
    
    .material-icons-round {
      font-size: 1.1rem;
    }
  }

  &__actions {
    display: flex;
    gap: $spacing-sm;
    margin-top: $spacing-lg;
    flex-wrap: wrap;

    @media (max-width: $breakpoint-sm) {
      gap: $spacing-xs;
      margin-top: $spacing-md;
      
      .btn {
        flex: 1;
        min-width: calc(50% - #{$spacing-xs});
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
  padding: $spacing-sm $spacing-md;
  border: 2px solid $border;
  border-radius: $border-radius;
  font-size: $font-size-base;
  font-family: $font-family-base;
  transition: all $transition-speed ease;
  background-color: white;
  color: $text;
  appearance: none;
  
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
