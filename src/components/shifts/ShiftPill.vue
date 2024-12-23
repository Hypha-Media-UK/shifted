<template>
  <div 
    class="shift-pill"
    :class="{
      'is-holiday': shift.isHoliday,
      'is-past': isPast,
      'is-morning': !shift.isHoliday && timeOfDayClass === 'is-morning',
      'is-afternoon': !shift.isHoliday && timeOfDayClass === 'is-afternoon',
      'is-evening': !shift.isHoliday && timeOfDayClass === 'is-evening',
      'is-night': !shift.isHoliday && timeOfDayClass === 'is-night'
    }"
    :title="`${formatShiftTime(shift.startTime)} - ${formatShiftTime(shift.endTime)}${shift.isHoliday ? ' (Holiday)' : ''}`"
  >
    <span 
      v-if="!isPast"
      class="shift-pill__time shift-pill__time--active" 
      @click.stop="$emit('edit')"
    >
      <span class="material-icons-round">{{ shift.isHoliday ? 'beach_access' : 'schedule' }}</span>
      {{ formatShiftTime(shift.startTime) }} - {{ formatShiftTime(shift.endTime) }}
    </span>
    <span 
      v-else
      class="shift-pill__time shift-pill__time--past"
    >
      <span class="material-icons-round">{{ shift.isHoliday ? 'beach_access' : 'schedule' }}</span>
      {{ formatShiftTime(shift.startTime) }} - {{ formatShiftTime(shift.endTime) }}
    </span>
    <button 
      v-if="!isPast"
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
  isPast?: boolean;
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
@use 'sass:color';
@use '../../styles/main' as *;

.shift-pill {
  padding: $spacing-xs $spacing-sm;
  border-radius: 8px;
  font-size: $font-size-sm;
  display: inline-flex;
  align-items: center;
  gap: $spacing-xs;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  background-color: rgba($text, 0.05) !important;
  border: 1px solid rgba($text, 0.1) !important;
  box-shadow: none !important;
  transform: translateY(0);
  overflow: hidden;
  min-width: 0;
  flex: 1;
  max-width: 150px;
  animation: pill-appear 0.5s cubic-bezier(0.34, 1.56, 0.64, 1);
  will-change: transform, opacity;
  transform-origin: center;

  @keyframes pill-appear {
    0% {
      opacity: 0;
      transform: translate3d(0, 0, 0) scale(0.8) translateY(4px);
    }
    60% {
      transform: translate3d(0, 0, 0) scale(1.02);
    }
    100% {
      opacity: 1;
      transform: translate3d(0, 0, 0) scale(1) translateY(0);
    }
  }

  &.is-morning {
    background-color: rgba($morning, 0.2) !important;
    border-color: rgba($morning, 0.3) !important;
    .shift-pill__time--active:hover {
      color: color.adjust($morning, $lightness: -35%) !important;
      .material-icons-round {
        color: color.adjust($morning, $lightness: -35%) !important;
      }
    }
  }

  &.is-afternoon {
    background-color: rgba($afternoon, 0.2) !important;
    border-color: rgba($afternoon, 0.3) !important;
    .shift-pill__time--active:hover {
      color: color.adjust($afternoon, $lightness: -35%) !important;
      .material-icons-round {
        color: color.adjust($afternoon, $lightness: -35%) !important;
      }
    }
  }

  &.is-evening {
    background-color: rgba($evening, 0.2) !important;
    border-color: rgba($evening, 0.3) !important;
    .shift-pill__time--active:hover {
      color: color.adjust($evening, $lightness: -35%) !important;
      .material-icons-round {
        color: color.adjust($evening, $lightness: -35%) !important;
      }
    }
  }

  &.is-night {
    background-color: rgba($night, 0.2) !important;
    border-color: rgba($night, 0.3) !important;
    .shift-pill__time--active:hover {
      color: color.adjust($night, $lightness: -35%) !important;
      .material-icons-round {
        color: color.adjust($night, $lightness: -35%) !important;
      }
    }
  }

  &.is-holiday {
    background-color: rgba($holiday, 0.25) !important;
    border-color: rgba($holiday, 0.4) !important;
    box-shadow: 0 1px 3px rgba($holiday, 0.2) !important;
    transform: translateY(-1px) !important;

    .shift-pill__time--active {
      color: $holiday !important;
      .material-icons-round {
        color: $holiday !important;
      }
      &:hover {
        color: color.adjust($holiday, $lightness: -10%) !important;
        .material-icons-round {
          color: color.adjust($holiday, $lightness: -10%) !important;
        }
      }
    }

    &:hover {
      box-shadow: 0 2px 4px rgba($holiday, 0.2) !important;
    }

    &:active {
      transform: translate3d(0, 0, 0) scale(0.98) !important;
      box-shadow: 0 1px 3px rgba($holiday, 0.2) !important;
    }
  }

  &.is-past {
    background-color: rgba($text, 0.05) !important;
    border-color: rgba($text, 0.1) !important;
    box-shadow: none !important;

    &.is-holiday {
      background-color: rgba($holiday, 0.1) !important;
      border-color: rgba($holiday, 0.2) !important;
      transform: translateY(0) !important;
      .shift-pill__time--past {
        color: rgba($holiday, 0.6) !important;
        .material-icons-round {
          color: rgba($holiday, 0.4) !important;
        }
      }
    }
  }

  &:not(.is-past):hover:not(.is-holiday) {
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1) !important;
    transform: translateY(-1px);
  }

  &:not(.is-past):active:not(.is-holiday) {
    transform: translate3d(0, 0, 0) scale(0.98);
    box-shadow: none !important;
  }

  &__time {
    display: inline-flex;
    align-items: center;
    gap: $spacing-xs;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    color: $text !important;
    font-weight: $font-weight-medium;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    min-width: 0;
    flex: 1;

    &--active {

      .material-icons-round {
        color: $text-light !important;
      }
    }

    &--past {
      cursor: default;
      color: rgba($text, 0.6) !important;

      .material-icons-round {
        color: rgba($text, 0.4) !important;
      }
    }

    .material-icons-round {
      font-size: 1rem;
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
    background: transparent !important;
    color: $text-light !important;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    padding: 0;
    transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
    flex-shrink: 0;

    .material-icons-round {
      font-size: 1rem;
    }

    &:hover {
      background: $danger !important;
      color: white !important;
    }
  }

  @media (max-width: $breakpoint-sm) {
    padding: $spacing-xs;
    font-size: $font-size-sm;
    max-width: 100px;
  }
}
</style>
