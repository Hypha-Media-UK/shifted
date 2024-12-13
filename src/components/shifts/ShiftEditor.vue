<template>
  <div class="shift-editor">
    <div class="form-group">
      <label>Start Time</label>
      <input 
        type="time" 
        class="form-control" 
        v-model="shift.startTime"
        :disabled="disabled"
      >
    </div>
    <div class="form-group">
      <label>End Time</label>
      <input 
        type="time" 
        class="form-control" 
        v-model="shift.endTime"
        :disabled="disabled"
      >
    </div>
    
    <div v-if="warning" class="shift-editor__warning">
      {{ warning }}
    </div>

    <div class="shift-editor__actions">
      <template v-if="isEditing">
        <button 
          class="btn btn-primary" 
          @click="$emit('update')"
          :disabled="!isValid || hasOverlap"
        >
          Update
        </button>
        <button class="btn btn-danger" @click="$emit('delete')">Delete</button>
        <button class="btn btn-secondary" @click="$emit('cancel')">Cancel</button>
      </template>
      <template v-else>
        <button 
          class="btn btn-primary" 
          @click="$emit('apply')"
          :disabled="!isValid || hasOverlap || disabled"
        >
          Apply Once
        </button>
        <button 
          class="btn btn-secondary" 
          @click="$emit('apply-and-copy')"
          :disabled="!isValid || hasOverlap || disabled"
        >
          Apply and Add to More
        </button>
        <button class="btn btn-secondary" @click="$emit('cancel')">Cancel</button>
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
@import '@/styles/abstracts/variables';

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

.form-group {
  margin-bottom: $spacing-md;
}

.form-control {
  width: 100%;
  padding: $spacing-sm;
  border: 1px solid $border;
  border-radius: $border-radius;
  font-size: $font-size-base;
  transition: border-color $transition-speed ease;
  
  &:focus {
    outline: none;
    border-color: $primary;
  }
}
</style>
