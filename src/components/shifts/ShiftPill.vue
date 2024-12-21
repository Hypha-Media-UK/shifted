<template>
  <div 
    class="shift-pill"
    :class="[timeOfDayClass, { 'is-holiday': shift.isHoliday }]"
    :title="`${formatShiftTime(shift.startTime)} - ${formatShiftTime(shift.endTime)}${shift.isHoliday ? ' (Holiday)' : ''}`"
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
import { computed } from 'vue';

interface ShiftTime {
  startTime: string;
  endTime: string;
  isHoliday?: boolean;
}

interface Shift extends ShiftTime {
  id: number;
}

const props = defineProps<{
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

const timeOfDayClass = computed(() => {
  if (props.shift.isHoliday) {
    return 'is-holiday';
  }

  const hour = parseInt(props.shift.startTime.split(':')[0]);
  
  if (hour >= 5 && hour < 12) {
    return 'is-morning';
  } else if (hour >= 12 && hour < 17) {
    return 'is-afternoon';
  } else if (hour >= 17 && hour < 22) {
    return 'is-evening';
  } else {
    return 'is-night';
  }
});
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

  &.is-morning {
    background-color: rgba($morning, 0.2);
    border-color: rgba($morning, 0.3);
    .shift-pill__time:hover {
      color: darken($morning, 35%);
      .material-icons-round {
        color: darken($morning, 35%);
      }
    }
  }

  &.is-afternoon {
    background-color: rgba($afternoon, 0.2);
    border-color: rgba($afternoon, 0.3);
    .shift-pill__time:hover {
      color: darken($afternoon, 35%);
      .material-icons-round {
        color: darken($afternoon, 35%);
      }
    }
  }

  &.is-evening {
    background-color: rgba($evening, 0.2);
    border-color: rgba($evening, 0.3);
    .shift-pill__time:hover {
      color: darken($evening, 35%);
      .material-icons-round {
        color: darken($evening, 35%);
      }
    }
  }

  &.is-night {
    background-color: rgba($night, 0.2);
    border-color: rgba($night, 0.3);
    .shift-pill__time:hover {
      color: darken($night, 35%);
      .material-icons-round {
        color: darken($night, 35%);
      }
    }
  }

  &.is-holiday {
    background-color: rgba($holiday, 0.2);
    border-color: rgba($holiday, 0.3);
    .shift-pill__time {
      color: darken($holiday, 20%);
      .material-icons-round {
        color: darken($holiday, 20%);
      }
      &:hover {
        color: darken($holiday, 35%);
        .material-icons-round {
          color: darken($holiday, 35%);
        }
      }
    }
  }

  &:hover {
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
