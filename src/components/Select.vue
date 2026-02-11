<template>
  <div :class="wrapperClasses">
    <label v-if="label" :for="id" class="select__label">
      {{ label }}
      <span v-if="required" class="select__required">*</span>
    </label>

    <div class="select__wrapper" ref="wrapperRef">
      <button
        :id="id"
        ref="triggerRef"
        type="button"
        :class="selectClasses"
        :disabled="disabled"
        @click="toggleDropdown"
        @blur="handleBlur"
      >
        <span class="select__value">
          <slot name="selected">
            {{ displayValue }}
          </slot>
        </span>
        <span class="select__arrow">
          <ChevronDownIcon />
        </span>
      </button>

      <Transition name="dropdown">
        <div v-if="isOpen" class="select__dropdown">
          <div
            v-for="option in filteredOptions"
            :key="option.value"
            :class="optionClasses(option)"
            @click="selectOption(option)"
            @mousedown.prevent
          >
            <slot name="option" v-bind="{ option }">
              {{ option.label }}
            </slot>
            <span v-if="option.value === modelValue" class="select__check">
              <CheckIcon />
            </span>
          </div>
          <div v-if="filteredOptions.length === 0" class="select__empty">
            {{ noResultsText }}
          </div>
        </div>
      </Transition>
    </div>

    <span v-if="error" class="select__error">{{ error }}</span>
    <span v-if="hint && !error" class="select__hint">{{ hint }}</span>
  </div>
</template>

<script setup lang="ts">
import { computed, ref, watch, onMounted, onUnmounted } from 'vue'

interface Option {
  value: string | number
  label: string
  disabled?: boolean
}

interface Props {
  id?: string
  modelValue?: string | number | null
  options: Option[]
  label?: string
  placeholder?: string
  error?: string
  hint?: string
  disabled?: boolean
  readonly?: boolean
  required?: boolean
  searchable?: boolean
  clearable?: boolean
  block?: boolean
  noResultsText?: string
}

const props = withDefaults(defineProps<Props>(), {
  id: '',
  modelValue: null,
  label: '',
  placeholder: 'Select...',
  error: '',
  hint: '',
  disabled: false,
  readonly: false,
  required: false,
  searchable: false,
  clearable: false,
  block: false,
  noResultsText: 'No results found'
})

const emit = defineEmits<{
  'update:modelValue': [value: string | number | null]
  'focus': [event: FocusEvent]
  'blur': [event: FocusEvent]
  'open': []
  'close': []
}>()

const isOpen = ref(false)
const searchQuery = ref('')
const wrapperRef = ref<HTMLElement>()
const triggerRef = ref<HTMLButtonElement>()

const uniqueId = computed(() => props.id || `select-${Math.random().toString(36).substr(2, 9)}`)

const wrapperClasses = computed(() => ({
  'select': true,
  'select--block': props.block,
  'select--error': !!props.error,
  'select--disabled': props.disabled
}))

const selectClasses = computed(() => [
  'select__trigger',
  {
    'select__trigger--error': !!props.error,
    'select__trigger--disabled': props.disabled,
    'select__trigger--open': isOpen.value
  }
])

const displayValue = computed(() => {
  if (props.modelValue === null || props.modelValue === undefined) {
    return props.placeholder
  }
  const option = props.options.find(o => o.value === props.modelValue)
  return option?.label || props.placeholder
})

const filteredOptions = computed(() => {
  if (!props.searchable || !searchQuery.value) {
    return props.options
  }
  const query = searchQuery.value.toLowerCase()
  return props.options.filter(o =>
    o.label.toLowerCase().includes(query)
  )
})

const optionClasses = (option: Option) => ({
  'select__option': true,
  'select__option--selected': option.value === props.modelValue,
  'select__option--disabled': option.disabled
})

const toggleDropdown = () => {
  if (props.disabled || props.readonly) return
  isOpen.value = !isOpen.value
  if (isOpen.value) {
    emit('open')
  } else {
    emit('close')
  }
}

const selectOption = (option: Option) => {
  if (option.disabled) return
  emit('update:modelValue', option.value)
  isOpen.value = false
  emit('close')
}

const handleBlur = (event: FocusEvent) => {
  // Delay to allow dropdown click to register
  setTimeout(() => {
    if (!wrapperRef.value?.contains(document.activeElement)) {
      isOpen.value = false
      emit('blur', event)
    }
  }, 150)
}

const closeDropdown = () => {
  isOpen.value = false
}

// Close on escape
const handleEscape = (event: KeyboardEvent) => {
  if (event.key === 'Escape' && isOpen.value) {
    closeDropdown()
  }
}

// Close on click outside
const handleClickOutside = (event: MouseEvent) => {
  if (wrapperRef.value && !wrapperRef.value.contains(event.target as Node)) {
    closeDropdown()
  }
}

onMounted(() => {
  document.addEventListener('keydown', handleEscape)
  document.addEventListener('click', handleClickOutside)
})

onUnmounted(() => {
  document.removeEventListener('keydown', handleEscape)
  document.removeEventListener('click', handleClickOutside)
})

defineExpose({
  open: () => { isOpen.value = true },
  close: closeDropdown,
  focus: () => triggerRef.value?.focus()
})
</script>

<script lang="ts">
const ChevronDownIcon = {
  template: `
    <svg width="16" height="16" viewBox="0 0 16 16" fill="currentColor">
      <path d="M4 6l4 4 4-4H4z"/>
    </svg>
  `
}

const CheckIcon = {
  template: `
    <svg width="16" height="16" viewBox="0 0 16 16" fill="currentColor">
      <path d="M13.5 3.5L6 11l-3.5-3.5L1 9l5 5 9-9-1.5-1.5z"/>
    </svg>
  `
}
</script>

<style scoped>
.select {
  display: inline-block;
  width: 100%;
  position: relative;
}

.select--block {
  display: flex;
  flex-direction: column;
}

.select__label {
  display: block;
  font-size: 0.875rem;
  font-weight: 500;
  color: #374151;
  margin-bottom: 0.375rem;
}

.select__required {
  color: #ef4444;
  margin-left: 0.25rem;
}

.select__wrapper {
  position: relative;
}

.select__trigger {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0.625rem 0.875rem;
  font-size: 0.875rem;
  line-height: 1.5;
  color: #111827;
  background-color: #ffffff;
  border: 1px solid #d1d5db;
  border-radius: 0.5rem;
  cursor: pointer;
  transition: all 0.2s ease;
  text-align: left;
}

.select__trigger:focus {
  outline: none;
  border-color: #3b82f6;
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
}

.select__trigger--error {
  border-color: #ef4444;
}

.select__trigger--error:focus {
  box-shadow: 0 0 0 3px rgba(239, 68, 68, 0.1);
}

.select__trigger--disabled {
  background-color: #f3f4f6;
  color: #9ca3af;
  cursor: not-allowed;
}

.select__value {
  flex: 1;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.select__arrow {
  display: flex;
  align-items: center;
  margin-left: 0.5rem;
  color: #6b7280;
  transition: transform 0.2s ease;
}

.select__trigger--open .select__arrow {
  transform: rotate(180deg);
}

.select__dropdown {
  position: absolute;
  top: calc(100% + 0.25rem);
  left: 0;
  right: 0;
  background-color: #ffffff;
  border: 1px solid #e5e7eb;
  border-radius: 0.5rem;
  box-shadow: 0 10px 15px rgba(0, 0, 0, 0.1);
  max-height: 15rem;
  overflow-y: auto;
  z-index: 100;
}

.select__option {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0.625rem 0.875rem;
  font-size: 0.875rem;
  cursor: pointer;
  transition: background-color 0.15s ease;
}

.select__option:hover:not(.select__option--disabled) {
  background-color: #f3f4f6;
}

.select__option--selected {
  background-color: #eff6ff;
  color: #3b82f6;
  font-weight: 500;
}

.select__option--disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.select__check {
  display: flex;
  align-items: center;
}

.select__empty {
  padding: 0.875rem;
  font-size: 0.875rem;
  color: #9ca3af;
  text-align: center;
}

.select__error {
  display: block;
  margin-top: 0.25rem;
  font-size: 0.75rem;
  color: #ef4444;
}

.select__hint {
  display: block;
  margin-top: 0.25rem;
  font-size: 0.75rem;
  color: #6b7280;
}

/* Dropdown Transition */
.dropdown-enter-active,
.dropdown-leave-active {
  transition: opacity 0.15s ease, transform 0.15s ease;
  transform-origin: top;
}

.dropdown-enter-from,
.dropdown-leave-to {
  opacity: 0;
  transform: scaleY(0.95);
}

/* Dark Mode */
@media (prefers-color-scheme: dark) {
  .select__label {
    color: #e5e7eb;
  }

  .select__trigger {
    color: #f9fafb;
    background-color: #1f2937;
    border-color: #4b5563;
  }

  .select__trigger:focus {
    border-color: #60a5fa;
    box-shadow: 0 0 0 3px rgba(96, 165, 250, 0.1);
  }

  .select__trigger--disabled {
    background-color: #374151;
  }

  .select__arrow {
    color: #9ca3af;
  }

  .select__dropdown {
    background-color: #1f2937;
    border-color: #374151;
  }

  .select__option:hover:not(.select__option--disabled) {
    background-color: #374151;
  }

  .select__option--selected {
    background-color: #1e3a5f;
  }
}
</style>
