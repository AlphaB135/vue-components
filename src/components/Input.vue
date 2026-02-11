<template>
  <div :class="wrapperClasses">
    <label v-if="label" :for="id" class="input__label">
      {{ label }}
      <span v-if="required" class="input__required">*</span>
    </label>

    <div class="input__wrapper">
      <span v-if="$slots.prefix" class="input__prefix">
        <slot name="prefix"></slot>
      </span>

      <input
        :id="id"
        ref="inputRef"
        :type="type"
        :value="modelValue"
        :placeholder="placeholder"
        :disabled="disabled"
        :readonly="readonly"
        :maxlength="maxlength"
        :class="inputClasses"
        @input="handleInput"
        @blur="handleBlur"
        @focus="handleFocus"
      />

      <span v-if="clearable && modelValue" class="input__clear" @click="handleClear">
        ×
      </span>

      <span v-if="$slots.suffix" class="input__suffix">
        <slot name="suffix"></slot>
      </span>
    </div>

    <span v-if="error" class="input__error">{{ error }}</span>
    <span v-if="hint && !error" class="input__hint">{{ hint }}</span>
  </div>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue'

interface Props {
  id?: string
  type?: 'text' | 'email' | 'password' | 'number' | 'tel' | 'url'
  modelValue?: string | number
  label?: string
  placeholder?: string
  error?: string
  hint?: string
  disabled?: boolean
  readonly?: boolean
  required?: boolean
  clearable?: boolean
  maxlength?: number
  block?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  type: 'text',
  modelValue: '',
  label: '',
  placeholder: '',
  error: '',
  hint: '',
  disabled: false,
  readonly: false,
  required: false,
  clearable: false,
  block: false
})

const emit = defineEmits<{
  'update:modelValue': [value: string]
  'focus': [event: FocusEvent]
  'blur': [event: FocusEvent]
  'clear': []
}>()

const inputRef = ref<HTMLInputElement>()

const uniqueId = computed(() => props.id || `input-${Math.random().toString(36).substr(2, 9)}`)

const wrapperClasses = computed(() => ({
  'input': true,
  'input--block': props.block,
  'input--error': !!props.error,
  'input--disabled': props.disabled,
  'input--readonly': props.readonly
}))

const inputClasses = computed(() => ({
  'input__field': true,
  'input__field--error': !!props.error,
  'input__field--disabled': props.disabled,
  'input__field--readonly': props.readonly
}))

const handleInput = (event: Event) => {
  const target = event.target as HTMLInputElement
  emit('update:modelValue', target.value)
}

const handleFocus = (event: FocusEvent) => {
  emit('focus', event)
}

const handleBlur = (event: FocusEvent) => {
  emit('blur', event)
}

const handleClear = () => {
  emit('update:modelValue', '')
  emit('clear')
}

const focus = () => {
  inputRef.value?.focus()
}

defineExpose({
  focus
})
</script>

<style scoped>
.input {
  display: inline-block;
  width: 100%;
}

.input--block {
  display: flex;
  flex-direction: column;
}

.input__label {
  display: block;
  font-size: 0.875rem;
  font-weight: 500;
  color: #374151;
  margin-bottom: 0.375rem;
}

.input__required {
  color: #ef4444;
  margin-left: 0.25rem;
}

.input__wrapper {
  position: relative;
  display: flex;
  align-items: center;
}

.input__field {
  width: 100%;
  padding: 0.625rem 0.875rem;
  font-size: 0.875rem;
  line-height: 1.5;
  color: #111827;
  background-color: #ffffff;
  border: 1px solid #d1d5db;
  border-radius: 0.5rem;
  transition: all 0.2s ease;
}

.input__field::placeholder {
  color: #9ca3af;
}

.input__field:focus {
  outline: none;
  border-color: #3b82f6;
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
}

.input__field--error {
  border-color: #ef4444;
}

.input__field--error:focus {
  border-color: #ef4444;
  box-shadow: 0 0 0 3px rgba(239, 68, 68, 0.1);
}

.input__field--disabled {
  background-color: #f3f4f6;
  color: #9ca3af;
  cursor: not-allowed;
}

.input__field--readonly {
  background-color: #f9fafb;
  cursor: default;
}

.input__prefix,
.input__suffix {
  position: absolute;
  display: flex;
  align-items: center;
  color: #6b7280;
}

.input__prefix {
  left: 0.875rem;
}

.input__suffix {
  right: 0.875rem;
}

.input__field:has(+ .input__suffix) {
  padding-right: 2.5rem;
}

.input__prefix + .input__field {
  padding-left: 2.5rem;
}

.input__clear {
  position: absolute;
  right: 0.875rem;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 1.25rem;
  height: 1.25rem;
  font-size: 1.125rem;
  color: #9ca3af;
  cursor: pointer;
  border-radius: 50%;
  transition: all 0.2s ease;
}

.input__clear:hover {
  background-color: #f3f4f6;
  color: #6b7280;
}

.input__error {
  display: block;
  margin-top: 0.25rem;
  font-size: 0.75rem;
  color: #ef4444;
}

.input__hint {
  display: block;
  margin-top: 0.25rem;
  font-size: 0.75rem;
  color: #6b7280;
}

/* Dark Mode */
@media (prefers-color-scheme: dark) {
  .input__label {
    color: #e5e7eb;
  }

  .input__field {
    color: #f9fafb;
    background-color: #1f2937;
    border-color: #4b5563;
  }

  .input__field::placeholder {
    color: #6b7280;
  }

  .input__field:focus {
    border-color: #60a5fa;
    box-shadow: 0 0 0 3px rgba(96, 165, 250, 0.1);
  }

  .input__field--disabled {
    background-color: #374151;
  }

  .input__field--readonly {
    background-color: #111827;
  }

  .input__prefix,
  .input__suffix {
    color: #9ca3af;
  }

  .input__clear:hover {
    background-color: #374151;
    color: #d1d5db;
  }
}
</style>
