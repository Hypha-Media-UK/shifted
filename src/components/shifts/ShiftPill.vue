<template>
  <div 
    class="shift-pill"
    :style="{ backgroundColor: `${accentColor}15` }"
    :title="`${formatShiftTime(shift.startTime)} - ${formatShiftTime(shift.endTime)}`"
  >
    <span class="shift-pill__time" @click.stop="$emit('edit')">
      <span class="material-icons-round">schedule</span>
      {{ formatShiftTime(shift.startTime) }} - {{ formatShiftTime(shift.endTime) }}
    </span>
    <button 
      class="shift-pill__delete" 
      @click.stop="$emit('delete')"
      title="Delete shift"
      aria-label="Delete shift"
    >
      <span class="material-icons-round">close</span>
    </button>
  </div>
</template>

<script setup lang="ts">
interface ShiftTime {
  startTime: string;
  endTime: string;
}

interface Shift extends ShiftTime {
  id: number;
}

defineProps<{
  shift: Shift;
  accentColor: string;
}>();

defineEmits<{
  (e: 'edit'): void;
  (e: 'delete'): void;
}>();

const formatShiftTime = (time: string) => {
  const [hours, minutes] = time.split(':');
  return `${hours}:${minutes}`;
};
</script>

<style lang="scss" scoped>
@import '../../styles/abstracts/variables';

.shift-pill {
  padding: $spacing-xs $spacing-sm;
  border-radius: $border-radius;
  font-size: $font-size-sm;
  display: inline-flex;
  align-items: center;
  gap: $spacing-xs;
  border: 1px solid transparent;
  transition: all $transition-speed ease;
  overflow: hidden;
  min-width: 0;
  flex: 1;
  max-width: 150px;

  &:hover {
    border-color: rgba($primary, 0.2);
    box-shadow: $shadow-sm;
  }

  &__time {
    display: inline-flex;
    align-items: center;
    gap: $spacing-xs;
    cursor: pointer;
    transition: all $transition-speed ease;
    color: $text;
    font-weight: $font-weight-medium;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    min-width: 0;
    flex: 1;

    .material-icons-round {
      font-size: 1rem;
      color: $text-light;
      flex-shrink: 0;
    }

    &:hover {
      color: $primary;

      .material-icons-round {
        color: $primary;
      }
    }

    @media (max-width: $breakpoint-sm) {
      .material-icons-round {
        display: none;
      }
    }
  }

  &__delete {
    width: 20px;
    height: 20px;
    min-width: 20px;
    border-radius: $border-radius-sm;
    border: none;
    background: transparent;
    color: $text-light;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    padding: 0;
    transition: all $transition-speed ease;
    flex-shrink: 0;

    .material-icons-round {
      font-size: 1rem;
    }

    &:hover {
      background: $danger;
      color: white;
    }
  }

  @media (max-width: $breakpoint-sm) {
    padding: $spacing-xs;
    font-size: $font-size-sm;
    max-width: 100px;
  }
}
</style>
