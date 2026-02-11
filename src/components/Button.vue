<template>
  <button
    :class="classes"
    :disabled="disabled || loading"
    @click="handleClick"
    :type="type"
  >
    <span v-if="loading" class="spinner"></span>
    <slot></slot>
  </button>
</template>

<script setup lang="ts">
import { computed } from 'vue'

interface Props {
  variant?: 'primary' | 'secondary' | 'danger' | 'ghost'
  size?: 'sm' | 'md' | 'lg'
  disabled?: boolean
  loading?: boolean
  type?: 'button' | 'submit' | 'reset'
  block?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  variant: 'primary',
  size: 'md',
  disabled: false,
  loading: false,
  type: 'button',
  block: false
})

const emit = defineEmits<{
  click: [event: MouseEvent]
}>()

const classes = computed(() => {
  return [
    'btn',
    `btn--${props.variant}`,
    `btn--${props.size}`,
    {
      'btn--disabled': props.disabled,
      'btn--loading': props.loading,
      'btn--block': props.block
    }
  ]
})

const handleClick = (event: MouseEvent) => {
  if (!props.disabled && !props.loading) {
    emit('click', event)
  }
}
</script>

<style scoped>
.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  border: none;
  border-radius: 0.5rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
  position: relative;
}

.btn:focus {
  outline: 2px solid currentColor;
  outline-offset: 2px;
}

.btn:disabled,
.btn--disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

/* Variants */
.btn--primary {
  background-color: #3b82f6;
  color: white;
}

.btn--primary:hover:not(:disabled) {
  background-color: #2563eb;
}

.btn--secondary {
  background-color: #6b7280;
  color: white;
}

.btn--secondary:hover:not(:disabled) {
  background-color: #4b5563;
}

.btn--danger {
  background-color: #ef4444;
  color: white;
}

.btn--danger:hover:not(:disabled) {
  background-color: #dc2626;
}

.btn--ghost {
  background-color: transparent;
  color: #3b82f6;
  border: 2px solid #3b82f6;
}

.btn--ghost:hover:not(:disabled) {
  background-color: #3b82f6;
  color: white;
}

/* Sizes */
.btn--sm {
  padding: 0.375rem 0.75rem;
  font-size: 0.875rem;
}

.btn--md {
  padding: 0.5rem 1rem;
  font-size: 1rem;
}

.btn--lg {
  padding: 0.75rem 1.5rem;
  font-size: 1.125rem;
}

.btn--block {
  width: 100%;
}

/* Loading Spinner */
.spinner {
  display: inline-block;
  width: 1em;
  height: 1em;
  border: 2px solid currentColor;
  border-radius: 50%;
  border-top-color: transparent;
  animation: spin 0.6s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

/* Dark Mode */
@media (prefers-color-scheme: dark) {
  .btn--primary {
    background-color: #60a5fa;
  }
  .btn--primary:hover:not(:disabled) {
    background-color: #3b82f6;
  }
}
</style>
