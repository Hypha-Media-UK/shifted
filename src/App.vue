<template>
  <Login v-if="!currentUser" @login="handleLogin" />
  <div v-else class="app">
    <header class="app__header">
      <div class="container">
        <div class="app__header-content">
          <h1>
            <span class="material-icons-round">calendar_today</span>
            Shifted
          </h1>

          <div class="user-controls">
            <span class="user-name">{{ currentUser }}</span>
            <button class="btn btn-icon" @click="handleLogout" title="Logout">
              <span class="material-icons-round">logout</span>
            </button>
          </div>
          
          <div class="app__header-controls">
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

            <div class="view-toggle">
              <div class="view-toggle__container">
                <button 
                  type="button"
                  class="view-toggle__btn"
                  :class="{ active: currentView === 'month' }"
                  @click="switchToMonth"
                >
                  Month
                </button>
                <button 
                  type="button"
                  class="view-toggle__btn"
                  :class="{ active: currentView === 'year' }"
                  @click="() => { currentView = 'year'; scrollToTop(); }"
                >
                  Year
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </header>

    <main class="app__main">
      <div class="container">
        <div class="app__title">
          <h2>{{ currentView === 'month' ? formattedMonth : currentYear }}</h2>
        </div>

        <Transition name="fade-scale" mode="out-in">
          <MonthView
            v-if="currentView === 'month'"
            :accent-color="accentColor"
            :selected-date="selectedDate"
            :initial-shifts="shifts"
            :current-date="currentDate"
            :clipboard-shift="clipboardShift"
            @update-shifts="updateShifts"
            @clear-selected-date="clearSelectedDate"
            @edit-shift="openShiftEditor"
            @copy-shift="(shift) => clipboardShift = shift"
            @clear-clipboard="() => clipboardShift = null"
          />
          <YearView
            v-else
            :shifts="shifts"
            :accent-color="accentColor"
            :current-date="currentDate"
            @select-date="handleDateSelect"
          />
        </Transition>

        <div class="danger-actions">
          <div class="danger-actions__container">
            <button 
              type="button"
              class="danger-actions__btn"
              @click="handleDeleteShifts"
            >
              <span class="material-icons-round">delete_sweep</span>
              Delete All Shifts
            </button>
            <button 
              type="button"
              class="danger-actions__btn"
              @click="handleDeleteAccount"
            >
              <span class="material-icons-round">delete_forever</span>
              Delete Account
            </button>
          </div>
        </div>
      </div>
    </main>

    <Teleport to="body">
      <ShiftEditor
        v-if="showShiftEditor"
        :shift="currentShift"
        :is-editing="isEditing"
        :disabled="false"
        :has-overlap="hasOverlap"
        :warning="validationWarning"
        @update="updateShift"
        @delete="deleteShift"
        @cancel="closeShiftEditor"
        @apply="applyShift"
        @apply-and-copy="copyShift"
        @toggle-holiday="toggleHoliday"
      />
    </Teleport>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch, computed } from 'vue';
import { format, addMonths, subMonths, addYears, subYears } from 'date-fns';
import MonthView from './components/shifts/MonthView.vue';
import YearView from './components/shifts/YearView.vue';
import Login from './components/auth/Login.vue';
import ShiftEditor from './components/shifts/ShiftEditor.vue';

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
const currentUser = ref<string | null>(null);
const clipboardShift = ref<ShiftTime | null>(null);

// Shift Editor State
const showShiftEditor = ref(false);
const isEditing = ref(false);
const editingShiftId = ref<number | null>(null);
const currentShift = ref<ShiftTime>({
  startTime: '',
  endTime: '',
  isHoliday: false
});

const hasOverlap = computed(() => {
  if (!currentShift.value.startTime || !currentShift.value.endTime) return false;
  
  const dateKey = format(selectedDate.value!, 'yyyy-MM-dd');
  const dayShifts = shifts.value[dateKey] || [];
  
  return dayShifts.some(shift => {
    if (isEditing.value && shift.id === editingShiftId.value) return false;
    return doShiftsOverlap(currentShift.value, shift);
  });
});

const validationWarning = computed(() => {
  if (!currentShift.value.startTime || !currentShift.value.endTime) return '';
  if (hasOverlap.value) return 'This shift overlaps with an existing shift';
  return '';
});

const timeToMinutes = (time: string): number => {
  const [hours, minutes] = time.split(':').map(Number);
  return hours * 60 + minutes;
};

const doShiftsOverlap = (shift1: ShiftTime, shift2: ShiftTime): boolean => {
  const start1 = timeToMinutes(shift1.startTime);
  const end1 = timeToMinutes(shift1.endTime);
  const start2 = timeToMinutes(shift2.startTime);
  const end2 = timeToMinutes(shift2.endTime);

  const adjustedEnd1 = end1 < start1 ? end1 + 24 * 60 : end1;
  const adjustedEnd2 = end2 < start2 ? end2 + 24 * 60 : end2;

  return (start1 < adjustedEnd2 && adjustedEnd1 > start2) ||
         (start2 < adjustedEnd1 && adjustedEnd2 > start1);
};

const openShiftEditor = (data: { shift?: Shift, date: Date }) => {
  selectedDate.value = data.date;
  
  if (data.shift) {
    isEditing.value = true;
    editingShiftId.value = data.shift.id;
    currentShift.value = { 
      startTime: data.shift.startTime,
      endTime: data.shift.endTime,
      isHoliday: data.shift.isHoliday
    };
  } else {
    isEditing.value = false;
    editingShiftId.value = null;
    currentShift.value = { startTime: '', endTime: '', isHoliday: false };
  }
  
  showShiftEditor.value = true;
};

const closeShiftEditor = () => {
  showShiftEditor.value = false;
  currentShift.value = { startTime: '', endTime: '', isHoliday: false };
  isEditing.value = false;
  editingShiftId.value = null;
};

const updateShift = () => {
  if (hasOverlap.value) return;
  
  const dateKey = format(selectedDate.value!, 'yyyy-MM-dd');
  const dayShifts = shifts.value[dateKey] || [];
  
  // Create new array with updated shift
  const updatedShifts = dayShifts.map(shift => {
    if (shift.id === editingShiftId.value) {
      // Create a completely new object to ensure reactivity
      return {
        ...shift,
        startTime: currentShift.value.startTime,
        endTime: currentShift.value.endTime,
        isHoliday: currentShift.value.isHoliday
      };
    }
    return shift;
  });
  
  // Force Vue to recognize the change by creating a new object
  shifts.value = {
    ...shifts.value,
    [dateKey]: [...updatedShifts]
  };
  
  closeShiftEditor();
};

const deleteShift = () => {
  const dateKey = format(selectedDate.value!, 'yyyy-MM-dd');
  const dayShifts = shifts.value[dateKey] || [];
  
  shifts.value = {
    ...shifts.value,
    [dateKey]: dayShifts.filter(shift => shift.id !== editingShiftId.value)
  };
  
  closeShiftEditor();
};

const applyShift = () => {
  if (hasOverlap.value) return;
  
  const dateKey = format(selectedDate.value!, 'yyyy-MM-dd');
  const dayShifts = shifts.value[dateKey] || [];
  
  const newShift: Shift = {
    id: Date.now(),
    startTime: currentShift.value.startTime,
    endTime: currentShift.value.endTime,
    isHoliday: currentShift.value.isHoliday
  };
  
  shifts.value = {
    ...shifts.value,
    [dateKey]: [...dayShifts, newShift]
  };
  
  clipboardShift.value = null;
  closeShiftEditor();
};

const copyShift = () => {
  if (hasOverlap.value) return;
  clipboardShift.value = { 
    startTime: currentShift.value.startTime,
    endTime: currentShift.value.endTime,
    isHoliday: currentShift.value.isHoliday
  };
  closeShiftEditor();
};

const toggleHoliday = (updatedShift?: ShiftTime) => {
  if (updatedShift) {
    currentShift.value = { ...updatedShift };
  } else {
    currentShift.value.isHoliday = !currentShift.value.isHoliday;
  }
  
  // For existing shifts, update immediately
  if (isEditing.value && editingShiftId.value) {
    const dateKey = format(selectedDate.value!, 'yyyy-MM-dd');
    const dayShifts = shifts.value[dateKey] || [];
    
    // Create new array with updated shift
    const updatedShifts = dayShifts.map(shift => {
      if (shift.id === editingShiftId.value) {
        return {
          ...shift,
          startTime: currentShift.value.startTime,
          endTime: currentShift.value.endTime,
          isHoliday: currentShift.value.isHoliday
        };
      }
      return shift;
    });
    
    // Force Vue to recognize the change
    shifts.value = {
      ...shifts.value,
      [dateKey]: [...updatedShifts]
    };
    
    // Close the editor after updating
    closeShiftEditor();
  }
};

// Computed
const currentYear = computed(() => currentDate.value.getFullYear());
const formattedMonth = computed(() => format(currentDate.value, 'MMM yyyy'));

// Load user from localStorage
const loadUser = () => {
  const savedUser = localStorage.getItem('shift-tracker-user');
  if (savedUser) {
    currentUser.value = savedUser;
  }
};

// Load data from localStorage
const loadData = () => {
  if (!currentUser.value) return;
  
  try {
    const savedData = localStorage.getItem(`${STORAGE_KEY}-${currentUser.value}`);
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
  if (!currentUser.value) return;

  try {
    const dataToSave = {
      shifts: shifts.value,
      selectedDate: selectedDate.value?.toISOString(),
      currentView: currentView.value,
      currentDate: currentDate.value.toISOString()
    };
    localStorage.setItem(`${STORAGE_KEY}-${currentUser.value}`, JSON.stringify(dataToSave));
  } catch (error) {
    console.error('Error saving data to localStorage:', error);
  }
};

// Delete all shifts but keep the account
const handleDeleteShifts = () => {
  if (!currentUser.value) return;
  
  if (confirm('Are you sure you want to delete all your shifts? This action cannot be undone.')) {
    // Clear shifts
    shifts.value = {};
    // Save empty shifts to localStorage
    saveData();
  }
};

// Delete user and their data
const handleDeleteAccount = () => {
  if (!currentUser.value) return;
  
  if (confirm('Are you sure you want to delete your account? This will permanently remove your account and all your shift data.')) {
    // Remove user's shift data
    localStorage.removeItem(`${STORAGE_KEY}-${currentUser.value}`);
    // Remove user
    localStorage.removeItem('shift-tracker-user');
    // Reset app state
    currentUser.value = null;
    shifts.value = {};
    selectedDate.value = undefined;
    currentView.value = 'month';
    currentDate.value = new Date();
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
  scrollToTop();
};

const clearSelectedDate = () => {
  selectedDate.value = undefined;
};

const switchToMonth = () => {
  selectedDate.value = undefined;
  currentView.value = 'month';
  scrollToTop();
};

const scrollToTop = () => {
  window.scrollTo({
    top: 0,
    behavior: 'smooth'
  });
};

const navigateBack = () => {
  if (currentView.value === 'month') {
    currentDate.value = subMonths(currentDate.value, 1);
  } else {
    currentDate.value = subYears(currentDate.value, 1);
  }
  scrollToTop();
};

const navigateForward = () => {
  if (currentView.value === 'month') {
    currentDate.value = addMonths(currentDate.value, 1);
  } else {
    currentDate.value = addYears(currentDate.value, 1);
  }
  scrollToTop();
};

// Handle login
const handleLogin = (username: string) => {
  currentUser.value = username;
  localStorage.setItem('shift-tracker-user', username);
  loadData();
};

// Handle logout
const handleLogout = () => {
  saveData(); // Save current user's data before logging out
  currentUser.value = null;
  localStorage.removeItem('shift-tracker-user');
  shifts.value = {};
  selectedDate.value = undefined;
  currentView.value = 'month';
  currentDate.value = new Date();
};

// Load user and data when component is mounted
onMounted(() => {
  loadUser();
  if (currentUser.value) {
    loadData();
  }
});
</script>

<style lang="scss">
@use './styles/main' as *;

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
      align-items: flex-start;
      justify-content: space-between;
      padding: 0 $spacing-md;

      @media (max-width: $breakpoint-sm) {
        padding: 0 $spacing-sm;
      }
    }
  }

  &__header-content {
    display: grid;
    grid-template-areas: "logo nav user";
    grid-template-columns: auto 1fr auto;
    gap: $spacing-md;
    width: 100%;
    position: relative;
    align-items: center;

    @media (max-width: 960px) {
      grid-template-areas: 
        "logo . user"
        "nav nav nav";
      grid-template-columns: auto 1fr auto;
      row-gap: $spacing-md;
    }

    @media (max-width: $breakpoint-sm) {
      row-gap: $spacing-sm;
    }

    h1 {
      grid-area: logo;
      font-size: $font-size-lg;
      font-weight: $font-weight-bold;
      margin: 0;
      color: white;
      display: flex;
      align-items: center;
      gap: $spacing-sm;
      white-space: nowrap;

      .material-icons-round {
        font-size: 1.5rem;
      }
    }
  }

  &__header-controls {
    grid-area: nav;
    display: flex;
    align-items: center;
    gap: $spacing-md;
    justify-content: flex-end;
    max-width: 600px;
    margin-left: auto;

    @media (max-width: 960px) {
      width: 100%;
    }

    @media (max-width: $breakpoint-sm) {
      gap: $spacing-sm;
    }

    > * {
      width: 240px;

      @media (max-width: $breakpoint-sm) {
        width: 50%;
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
      padding: 0 $spacing-md;

      @media (max-width: $breakpoint-sm) {
        padding: 0 $spacing-sm;
      }
    }
  }

  &__title {
    margin-bottom: $spacing-lg;
    padding: 0 $spacing-md;
    text-align: center;

    @media (max-width: $breakpoint-sm) {
      margin-bottom: $spacing-md;
      padding: 0 $spacing-sm;
    }

    h2 {
      font-size: $font-size-lg;
      font-weight: $font-weight-bold;
      color: $text;
      margin: 0;
    }
  }
}

.user-controls {
  grid-area: user;
  display: flex;
  align-items: center;
  gap: $spacing-sm;
  background: rgba(white, 0.1);
  padding: $spacing-xs $spacing-lg;
  border-radius: $border-radius;
  justify-self: end;

  @media (max-width: $breakpoint-sm) {
    padding: $spacing-xs $spacing-md;
  }

  .user-name {
    color: white;
    font-weight: $font-weight-medium;
    font-size: $font-size-sm;
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

.date-navigation {
  display: flex;
  align-items: center;
  gap: $spacing-sm;
  background: rgba(white, 0.1);
  padding: $spacing-xs;
  border-radius: $border-radius;

  @media (max-width: $breakpoint-sm) {
    justify-content: center;
  }

  &__title {
    font-size: $font-size-base;
    font-weight: $font-weight-bold;
    margin: 0;
    color: white;
    min-width: 100px;
    text-align: center;

    @media (max-width: $breakpoint-sm) {
      font-size: $font-size-sm;
      min-width: 90px;
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

  @media (max-width: $breakpoint-sm) {
    min-width: 0;
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

.danger-actions {
  margin-top: $spacing-xl;
  position: relative;

  @media (max-width: $breakpoint-sm) {
    min-width: 0;
  }

  &__container {
    display: flex;
    background: rgba($danger, 0.1);
    padding: $spacing-xs;
    border-radius: $border-radius;
    gap: $spacing-xs;
    width: 100%;
    max-width: 600px;
    margin: 0 auto;
  }

  &__btn {
    flex: 1;
    padding: $spacing-sm $spacing-lg;
    border: none;
    border-radius: $border-radius-sm;
    background: transparent;
    color: rgba($danger, 0.8);
    font-size: $font-size-base;
    font-weight: $font-weight-medium;
    cursor: pointer;
    transition: all $transition-speed ease;
    white-space: nowrap;
    min-width: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: $spacing-sm;

    @media (max-width: $breakpoint-sm) {
      padding: $spacing-sm;
    }

    .material-icons-round {
      font-size: 1.2rem;
    }

    &:hover {
      color: $danger;
      background: rgba($danger, 0.1);
    }

    &.active {
      background: $danger;
      color: white;
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
</style>
