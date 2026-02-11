<template>
  <div :class="cardClasses">
    <div v-if="$slots.header || header" class="card__header">
      <slot name="header">
        <h3 class="card__title">{{ header }}</h3>
      </slot>
    </div>

    <div class="card__body">
      <slot></slot>
    </div>

    <div v-if="$slots.footer" class="card__footer">
      <slot name="footer"></slot>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'

interface Props {
  header?: string
  variant?: 'default' | 'bordered' | 'shadow' | 'flat'
  hover?: boolean
  clickable?: boolean
  padding?: 'none' | 'sm' | 'md' | 'lg'
}

const props = withDefaults(defineProps<Props>(), {
  header: '',
  variant: 'default',
  hover: false,
  clickable: false,
  padding: 'md'
})

const emit = defineEmits<{
  click: [event: MouseEvent]
}>()

const cardClasses = computed(() => [
  'card',
  `card--${props.variant}`,
  `card--padding-${props.padding}`,
  {
    'card--hover': props.hover,
    'card--clickable': props.clickable
  }
])

const handleClick = (event: MouseEvent) => {
  if (props.clickable) {
    emit('click', event)
  }
}
</script>

<style scoped>
.card {
  background-color: #ffffff;
  border-radius: 0.75rem;
  transition: all 0.2s ease;
  overflow: hidden;
}

.card--bordered {
  border: 1px solid #e5e7eb;
}

.card--shadow {
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1), 0 1px 2px rgba(0, 0, 0, 0.06);
}

.card--shadow:hover {
  box-shadow: 0 10px 15px rgba(0, 0, 0, 0.1), 0 4px 6px rgba(0, 0, 0, 0.05);
}

.card--flat {
  box-shadow: none;
  border: none;
  background-color: transparent;
}

.card--hover:hover {
  transform: translateY(-2px);
}

.card--clickable {
  cursor: pointer;
}

.card--clickable:hover {
  border-color: #3b82f6;
}

.card__header {
  padding: 1rem 1.25rem;
  border-bottom: 1px solid #f3f4f6;
}

.card__title {
  font-size: 1.125rem;
  font-weight: 600;
  color: #111827;
  margin: 0;
}

.card__body {
  padding: 1.25rem;
}

.card--padding-none .card__body {
  padding: 0;
}

.card--padding-sm .card__body {
  padding: 0.75rem;
}

.card--padding-md .card__body {
  padding: 1.25rem;
}

.card--padding-lg .card__body {
  padding: 1.75rem;
}

.card__footer {
  padding: 1rem 1.25rem;
  border-top: 1px solid #f3f4f6;
  background-color: #f9fafb;
  display: flex;
  gap: 0.5rem;
  justify-content: flex-end;
}

/* Dark Mode */
@media (prefers-color-scheme: dark) {
  .card {
    background-color: #1f2937;
  }

  .card--bordered {
    border-color: #374151;
  }

  .card__header {
    border-bottom-color: #374151;
  }

  .card__title {
    color: #f9fafb;
  }

  .card__footer {
    border-top-color: #374151;
    background-color: #111827;
  }
}
</style>
