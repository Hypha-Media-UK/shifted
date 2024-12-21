<template>
  <div class="app">
    <header class="app__header">
      <div class="container">
        <h1>
          <span class="material-icons-round">calendar_today</span>
          Shift Tracker
        </h1>
        <div class="view-toggle">
          <div class="view-toggle__container">
            <button 
              type="button"
              class="view-toggle__btn"
              :class="{ active: currentView === 'month' }"
              @click="switchToMonth"
            >
              Month View
            </button>
            <button 
              type="button"
              class="view-toggle__btn"
              :class="{ active: currentView === 'year' }"
              @click="currentView = 'year'"
            >
              Year View
            </button>
          </div>
        </div>
      </div>
    </header>

    <main class="app__main">
      <div class="container">
        <Transition name="fade-scale" mode="out-in">
          <MonthView
            v-if="currentView === 'month'"
            :accent-color="accentColor"
            :selected-date="selectedDate"
            :initial-shifts="shifts"
            @update-shifts="updateShifts"
            @clear-selected-date="clearSelectedDate"
          />
          <YearView
            v-else
            :shifts="shifts"
            :accent-color="accentColor"
            @select-date="handleDateSelect"
          />
        </Transition>
      </div>
    </main>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from 'vue';
import MonthView from './components/shifts/MonthView.vue';
import YearView from './components/shifts/YearView.vue';

interface ShiftTime {
  startTime: string;
  endTime: string;
  isHoliday?: boolean;
}

interface Shift extends ShiftTime {
  id: number;
}

interface ShiftMap {
  [key: string]: Shift[];
}

// Constants
const STORAGE_KEY = 'shift-tracker-data';
const accentColor = '#7c3aed';

// State
const currentView = ref<'month' | 'year'>('month');
const shifts = ref<ShiftMap>({});
const selectedDate = ref<Date | undefined>();

// Load data from localStorage
const loadData = () => {
  try {
    const savedData = localStorage.getItem(STORAGE_KEY);
    if (savedData) {
      const parsedData = JSON.parse(savedData);
      shifts.value = parsedData.shifts || {};
      
      if (parsedData.selectedDate) {
        selectedDate.value = new Date(parsedData.selectedDate);
      }
      
      if (parsedData.currentView) {
        currentView.value = parsedData.currentView;
      }
    }
  } catch (error) {
    console.error('Error loading data from localStorage:', error);
  }
};

// Save data to localStorage
const saveData = () => {
  try {
    const dataToSave = {
      shifts: shifts.value,
      selectedDate: selectedDate.value?.toISOString(),
      currentView: currentView.value
    };
    localStorage.setItem(STORAGE_KEY, JSON.stringify(dataToSave));
  } catch (error) {
    console.error('Error saving data to localStorage:', error);
  }
};

// Watch for changes in state and save to localStorage
watch([shifts, selectedDate, currentView], () => {
  saveData();
}, { deep: true });

// Methods
const updateShifts = (newShifts: ShiftMap) => {
  shifts.value = newShifts;
};

const handleDateSelect = (date: Date) => {
  selectedDate.value = date;
  currentView.value = 'month';
};

const clearSelectedDate = () => {
  selectedDate.value = undefined;
};

const switchToMonth = () => {
  selectedDate.value = undefined;
  currentView.value = 'month';
};

// Load data when component is mounted
onMounted(() => {
  loadData();
});
</script>

<style lang="scss">
@import './styles/main.scss';

.app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;

  &__header {
    background: $gradient-primary;
    box-shadow: $shadow-md;
    padding: $spacing-lg 0;
    margin-bottom: $spacing-xl;
    position: sticky;
    top: 0;
    z-index: 100;

    @media (max-width: $breakpoint-sm) {
      padding: $spacing-md 0;
      margin-bottom: $spacing-lg;
    }

    .container {
      display: flex;
      align-items: center;
      justify-content: space-between;
      gap: $spacing-md;

      @media (max-width: $breakpoint-sm) {
        flex-direction: column;
        align-items: stretch;
      }
    }

    h1 {
      font-size: $font-size-xl;
      font-weight: $font-weight-bold;
      margin: 0;
      color: white;
      display: flex;
      align-items: center;
      gap: $spacing-sm;

      .material-icons-round {
        font-size: 1.75rem;
      }

      @media (max-width: $breakpoint-sm) {
        justify-content: center;
        margin-bottom: $spacing-sm;
      }
    }
  }

  &__main {
    flex: 1;
    padding: $spacing-md 0 $spacing-xl;

    @media (max-width: $breakpoint-sm) {
      padding: $spacing-sm 0 $spacing-lg;
    }

    .container {
      @media (max-width: $breakpoint-sm) {
        padding: 0 $spacing-sm;
      }
    }
  }
}

.view-toggle {
  position: relative;
  width: 300px;

  @media (max-width: $breakpoint-sm) {
    width: 100%;
  }

  &__container {
    display: flex;
    background: rgba(white, 0.1);
    padding: $spacing-xs;
    border-radius: $border-radius;
    gap: $spacing-xs;
    width: 100%;
  }

  &__btn {
    flex: 1;
    padding: $spacing-sm $spacing-lg;
    border: none;
    border-radius: $border-radius-sm;
    background: transparent;
    color: rgba(white, 0.8);
    font-size: $font-size-base;
    font-weight: $font-weight-medium;
    cursor: pointer;
    transition: all $transition-speed ease;
    white-space: nowrap;
    min-width: 0;

    @media (max-width: $breakpoint-sm) {
      padding: $spacing-sm;
    }

    &:hover {
      color: white;
      background: rgba(white, 0.1);
    }

    &.active {
      background: white;
      color: $primary;
      box-shadow: $shadow-sm;

      &:hover {
        transform: translateY(-1px);
        box-shadow: $shadow-md;
      }

      &:active {
        transform: translateY(0);
      }
    }
  }
}

.fade-scale-enter-active,
.fade-scale-leave-active {
  transition: all $transition-speed $transition-bounce;
}

.fade-scale-enter-from,
.fade-scale-leave-to {
  opacity: 0;
  transform: scale(0.98) translateY(10px);
}
</style>
