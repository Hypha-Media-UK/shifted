<template>
  <div class="app">
    <header class="app__header">
      <div class="container">
        <div class="app__header-content">
          <h1>
            <span class="material-icons-round">calendar_today</span>
            Shift Tracker
          </h1>
          
          <div class="app__header-controls">
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

            <div class="date-navigation">
              <button class="btn btn-icon" @click="navigateBack">
                <span class="material-icons-round">chevron_left</span>
              </button>
              <h2 class="date-navigation__title">
                {{ currentView === 'month' ? formattedMonth : currentYear }}
              </h2>
              <button class="btn btn-icon" @click="navigateForward">
                <span class="material-icons-round">chevron_right</span>
              </button>
            </div>
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
            :current-date="currentDate"
            @update-shifts="updateShifts"
            @clear-selected-date="clearSelectedDate"
          />
          <YearView
            v-else
            :shifts="shifts"
            :accent-color="accentColor"
            :current-date="currentDate"
            @select-date="handleDateSelect"
          />
        </Transition>
      </div>
    </main>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch, computed } from 'vue';
import { format, addMonths, subMonths, addYears, subYears } from 'date-fns';
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
const currentDate = ref(new Date());

// Computed
const currentYear = computed(() => currentDate.value.getFullYear());
const formattedMonth = computed(() => format(currentDate.value, 'MMMM yyyy'));

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

      if (parsedData.currentDate) {
        currentDate.value = new Date(parsedData.currentDate);
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
      currentView: currentView.value,
      currentDate: currentDate.value.toISOString()
    };
    localStorage.setItem(STORAGE_KEY, JSON.stringify(dataToSave));
  } catch (error) {
    console.error('Error saving data to localStorage:', error);
  }
};

// Watch for changes in state and save to localStorage
watch([shifts, selectedDate, currentView, currentDate], () => {
  saveData();
}, { deep: true });

// Methods
const updateShifts = (newShifts: ShiftMap) => {
  shifts.value = newShifts;
};

const handleDateSelect = (date: Date) => {
  selectedDate.value = date;
  currentView.value = 'month';
  currentDate.value = date;
};

const clearSelectedDate = () => {
  selectedDate.value = undefined;
};

const switchToMonth = () => {
  selectedDate.value = undefined;
  currentView.value = 'month';
};

const navigateBack = () => {
  if (currentView.value === 'month') {
    currentDate.value = subMonths(currentDate.value, 1);
  } else {
    currentDate.value = subYears(currentDate.value, 1);
  }
};

const navigateForward = () => {
  if (currentView.value === 'month') {
    currentDate.value = addMonths(currentDate.value, 1);
  } else {
    currentDate.value = addYears(currentDate.value, 1);
  }
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
  }

  &__header-content {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: $spacing-lg;
    width: 100%;

    @media (max-width: $breakpoint-sm) {
      flex-direction: column;
      gap: $spacing-md;
    }

    h1 {
      font-size: $font-size-xl;
      font-weight: $font-weight-bold;
      margin: 0;
      color: white;
      display: flex;
      align-items: center;
      gap: $spacing-sm;
      white-space: nowrap;

      .material-icons-round {
        font-size: 1.75rem;
      }

      @media (max-width: $breakpoint-sm) {
        justify-content: center;
        margin-bottom: $spacing-sm;
      }
    }
  }

  &__header-controls {
    display: flex;
    align-items: center;
    gap: $spacing-lg;
    flex: 1;
    justify-content: flex-end;

    @media (max-width: $breakpoint-sm) {
      flex-direction: column;
      gap: $spacing-md;
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

.date-navigation {
  display: flex;
  align-items: center;
  gap: $spacing-sm;
  background: rgba(white, 0.1);
  padding: $spacing-xs;
  border-radius: $border-radius;

  @media (max-width: $breakpoint-sm) {
    width: 100%;
    justify-content: center;
  }

  &__title {
    font-size: $font-size-lg;
    font-weight: $font-weight-bold;
    margin: 0;
    color: white;
    min-width: 180px;
    text-align: center;

    @media (max-width: $breakpoint-sm) {
      font-size: $font-size-base;
      min-width: 140px;
    }
  }

  .btn-icon {
    width: 32px;
    height: 32px;
    padding: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    background: transparent;
    border: none;
    color: rgba(white, 0.8);
    cursor: pointer;
    transition: all $transition-speed ease;

    &:hover {
      color: white;
      background: rgba(white, 0.1);
    }

    .material-icons-round {
      font-size: 1.25rem;
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
