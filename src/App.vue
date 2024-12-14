<template>
  <div class="app">
    <header class="app__header">
      <div class="container">
        <h1>Shift Tracker</h1>
        <div class="view-toggle">
          <button 
            class="btn" 
            :class="{ 'btn-primary': currentView === 'month', 'btn-secondary': currentView === 'year' }"
            @click="switchToMonth"
          >
            Month View
          </button>
          <button 
            class="btn" 
            :class="{ 'btn-primary': currentView === 'year', 'btn-secondary': currentView === 'month' }"
            @click="currentView = 'year'"
          >
            Year View
          </button>
        </div>
      </div>
    </header>

    <main class="app__main">
      <div class="container">
        <Transition name="fade" mode="out-in">
          <MonthView
            v-if="currentView === 'month'"
            :accent-color="accentColor"
            :selected-date="selectedDate"
            :initial-shifts="shifts"
            @update-shifts="updateShifts"
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
import { ref } from 'vue';
import MonthView from './components/shifts/MonthView.vue';
import YearView from './components/shifts/YearView.vue';

interface ShiftTime {
  startTime: string;
  endTime: string;
}

interface Shift extends ShiftTime {
  id: number;
}

interface ShiftMap {
  [key: string]: Shift[];
}

const currentView = ref<'month' | 'year'>('month');
const shifts = ref<ShiftMap>({});
const selectedDate = ref<Date | undefined>();
const accentColor = '#4A90E2';

const updateShifts = (newShifts: ShiftMap) => {
  shifts.value = newShifts;
};

const handleDateSelect = (date: Date) => {
  selectedDate.value = date;
  currentView.value = 'month';
};

const switchToMonth = () => {
  selectedDate.value = undefined;
  currentView.value = 'month';
};
</script>

<style lang="scss">
@import './styles/main.scss';

.app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;

  &__header {
    background: white;
    box-shadow: $box-shadow;
    padding: $spacing-md 0;
    margin-bottom: $spacing-xl;

    .container {
      display: flex;
      align-items: center;
      justify-content: space-between;
    }

    h1 {
      font-size: 1.5rem;
      font-weight: 600;
      margin: 0;
      color: $text;
    }
  }

  &__main {
    flex: 1;
    padding: $spacing-md 0 $spacing-xl;
  }
}

.view-toggle {
  display: flex;
  gap: $spacing-sm;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity $transition-speed ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 $spacing-md;
}
</style>
