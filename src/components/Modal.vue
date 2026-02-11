<template>
  <Teleport to="body">
    <Transition name="modal">
      <div v-if="modelValue" class="modal-overlay" @click="handleOverlayClick">
        <div
          :class="modalClasses"
          @click.stop
          role="dialog"
          :aria-modal="true"
          :aria-labelledby="titleId"
        >
          <div v-if="!hideHeader" class="modal__header">
            <slot name="header">
              <h3 :id="titleId" class="modal__title">{{ title }}</h3>
            </slot>
            <button
              v-if="closable"
              class="modal__close"
              @click="handleClose"
              aria-label="Close"
            >
              <CloseIcon />
            </button>
          </div>

          <div :class="bodyClasses">
            <slot></slot>
          </div>

          <div v-if="$slots.footer" class="modal__footer">
            <slot name="footer"></slot>
          </div>
        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<script setup lang="ts">
import { computed, ref, watch } from 'vue'

interface Props {
  modelValue: boolean
  title?: string
  size?: 'sm' | 'md' | 'lg' | 'xl' | 'full'
  closable?: boolean
  hideHeader?: boolean
  closeOnOverlay?: boolean
  scrollLock?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  title: '',
  size: 'md',
  closable: true,
  hideHeader: false,
  closeOnOverlay: true,
  scrollLock: true
})

const emit = defineEmits<{
  'update:modelValue': [value: boolean]
  'open': []
  'close': []
}>()

const titleId = ref(`modal-title-${Math.random().toString(36).substr(2, 9)}`)

const modalClasses = computed(() => [
  'modal',
  `modal--${props.size}`
])

const bodyClasses = computed(() => [
  'modal__body',
  { 'modal__body--no-header': props.hideHeader }
])

const handleClose = () => {
  emit('update:modelValue', false)
  emit('close')
}

const handleOverlayClick = () => {
  if (props.closeOnOverlay) {
    handleClose()
  }
}

// Handle scroll lock
watch(() => props.modelValue, (isOpen) => {
  if (props.scrollLock) {
    if (isOpen) {
      document.body.style.overflow = 'hidden'
      emit('open')
    } else {
      document.body.style.overflow = ''
    }
  }
})

// Clean up scroll lock on unmount
const cleanupScrollLock = () => {
  document.body.style.overflow = ''
}

// Close on Escape key
const handleEscape = (event: KeyboardEvent) => {
  if (event.key === 'Escape' && props.modelValue && props.closable) {
    handleClose()
  }
}

if (typeof window !== 'undefined') {
  window.addEventListener('keydown', handleEscape)
  window.addEventListener('beforeunload', cleanupScrollLock)
}

defineExpose({
  close: handleClose
})
</script>

<script lang="ts">
// CloseIcon component
const CloseIcon = {
  template: `
    <svg width="20" height="20" viewBox="0 0 20 20" fill="currentColor">
      <path d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z"/>
    </svg>
  `
}
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  inset: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  padding: 1rem;
}

.modal {
  background-color: #ffffff;
  border-radius: 0.75rem;
  box-shadow: 0 20px 25px rgba(0, 0, 0, 0.1), 0 10px 10px rgba(0, 0, 0, 0.04);
  max-height: calc(100vh - 2rem);
  display: flex;
  flex-direction: column;
}

.modal--sm {
  width: 100%;
  max-width: 24rem;
}

.modal--md {
  width: 100%;
  max-width: 32rem;
}

.modal--lg {
  width: 100%;
  max-width: 48rem;
}

.modal--xl {
  width: 100%;
  max-width: 64rem;
}

.modal--full {
  width: 100%;
  max-width: 100%;
  height: calc(100vh - 2rem);
  max-height: calc(100vh - 2rem);
}

.modal__header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 1.25rem;
  border-bottom: 1px solid #e5e7eb;
}

.modal__title {
  font-size: 1.25rem;
  font-weight: 600;
  color: #111827;
  margin: 0;
}

.modal__close {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 2rem;
  height: 2rem;
  border: none;
  background: none;
  color: #6b7280;
  cursor: pointer;
  border-radius: 0.375rem;
  transition: all 0.2s ease;
}

.modal__close:hover {
  background-color: #f3f4f6;
  color: #111827;
}

.modal__body {
  padding: 1.25rem;
  overflow-y: auto;
  flex: 1;
}

.modal__body--no-header {
  border-top-left-radius: 0.75rem;
  border-top-right-radius: 0.75rem;
}

.modal__footer {
  padding: 1rem 1.25rem;
  border-top: 1px solid #e5e7eb;
  display: flex;
  gap: 0.5rem;
  justify-content: flex-end;
  background-color: #f9fafb;
  border-bottom-left-radius: 0.75rem;
  border-bottom-right-radius: 0.75rem;
}

/* Transitions */
.modal-enter-active,
.modal-leave-active {
  transition: opacity 0.2s ease;
}

.modal-enter-active .modal,
.modal-leave-active .modal {
  transition: transform 0.2s ease, opacity 0.2s ease;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}

.modal-enter-from .modal,
.modal-leave-to .modal {
  transform: scale(0.95);
  opacity: 0;
}

/* Dark Mode */
@media (prefers-color-scheme: dark) {
  .modal {
    background-color: #1f2937;
  }

  .modal__title {
    color: #f9fafb;
  }

  .modal__header {
    border-bottom-color: #374151;
  }

  .modal__close {
    color: #9ca3af;
  }

  .modal__close:hover {
    background-color: #374151;
    color: #f9fafb;
  }

  .modal__footer {
    background-color: #111827;
    border-top-color: #374151;
  }
}
</style>
