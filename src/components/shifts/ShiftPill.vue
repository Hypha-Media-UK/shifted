<template>
  <div 
    class="shift-pill"
    :style="{ backgroundColor: accentColor + '33' }"
  >
    <span @click.stop="$emit('edit')">
      {{ formatShiftTime(shift.startTime) }} - {{ formatShiftTime(shift.endTime) }}
    </span>
    <button 
      class="shift-pill__delete" 
      @click.stop="$emit('delete')"
      title="Quick delete"
    >
      ×
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
@import '@/styles/abstracts/variables';

.shift-pill {
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

  &__delete {
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
}
</style>
