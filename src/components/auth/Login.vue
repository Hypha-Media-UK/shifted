<template>
  <div class="login">
    <div class="login__container">
      <h2>
        <span class="material-icons-round">calendar_today</span>
        Shift Tracker
      </h2>
      
      <form @submit.prevent="handleLogin" class="login__form">
        <div class="form-group">
          <label class="form-label">Username</label>
          <input 
            type="text" 
            class="form-control" 
            v-model="username"
            placeholder="Enter username"
            required
            autocomplete="username"
          >
        </div>
        
        <button type="submit" class="btn btn-primary btn-wide">
          <span class="material-icons-round">login</span>
          Login
        </button>
      </form>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const emit = defineEmits<{
  (e: 'login', username: string): void;
}>();

const username = ref('');

const handleLogin = () => {
  if (username.value.trim()) {
    emit('login', username.value.trim());
  }
};
</script>

<style lang="scss" scoped>
@use 'sass:color';
@use '../../styles/main' as *;

.login {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: $spacing-lg;
  background: $gradient-primary;

  &__container {
    background: rgba(white, 0.98);
    padding: $spacing-xl;
    border-radius: 16px;
    width: 100%;
    max-width: 400px;
    box-shadow: $shadow-lg;
    backdrop-filter: blur(8px);

    h2 {
      font-size: $font-size-xl;
      font-weight: $font-weight-bold;
      color: $text;
      margin: 0 0 $spacing-xl;
      display: flex;
      align-items: center;
      gap: $spacing-sm;
      justify-content: center;

      .material-icons-round {
        font-size: 1.75rem;
        color: $primary;
      }
    }
  }

  &__form {
    display: flex;
    flex-direction: column;
    gap: $spacing-lg;
  }
}

.form-group {
  margin: 0;
}

.form-label {
  display: block;
  margin-bottom: $spacing-xs;
  color: rgba($text, 0.8);
  font-weight: $font-weight-medium;
  font-size: $font-size-sm;
  letter-spacing: 0.01em;
}

.form-control {
  width: 100%;
  padding: $spacing-sm $spacing-md;
  border: 1px solid rgba($border, 0.8);
  border-radius: 10px;
  font-size: $font-size-base;
  font-family: $font-family-base;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  background-color: rgba(white, 0.8);
  color: $text;
  appearance: none;
  height: 42px;
  letter-spacing: 0.01em;
  
  &:hover {
    border-color: rgba($text-light, 0.6);
    background-color: rgba(white, 0.9);
  }
  
  &:focus {
    outline: none;
    border-color: $primary;
    background-color: white;
    box-shadow: 0 2px 4px rgba($primary, 0.1);
  }
}

.btn {
  padding: $spacing-sm $spacing-lg;
  border-radius: 999px;
  border: none;
  font-size: $font-size-base;
  font-weight: $font-weight-medium;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: $spacing-xs;
  -webkit-tap-highlight-color: transparent;

  &.btn-wide {
    width: 100%;
  }

  &:active {
    transform: scale(0.98);
  }
}

.btn-primary {
  background: $primary;
  color: white;
  box-shadow: 0 1px 3px rgba($primary, 0.3);

  &:hover {
    background: color.adjust($primary, $lightness: -5%);
    box-shadow: 0 2px 4px rgba($primary, 0.4);
    transform: translateY(-1px);
  }

  .material-icons-round {
    font-size: 1.2rem;
  }
}
</style>
