# Vue Component Library

A collection of production-ready Vue 3 + TypeScript components.

## Components

### 1. Button
- Multiple variants (primary, secondary, danger, ghost)
- Sizes (sm, md, lg)
- Loading state
- Disabled state

### 2. Input
- Text, email, password, number types
- Validation support
- Error states
- Icons
- Clearable

### 3. Modal
- Backdrop
- Close on backdrop click
- Animation
- Custom content slot

### 4. Card
- Header, body, footer slots
- Hover effects
- Border variants

### 5. Dropdown
- Click or hover trigger
- Keyboard navigation
- Custom content

### 6. Loader
- Spinner
- Skeleton screen
- Progress bar

### 7. Toggle
- On/off states
- Custom colors
- Disabled state

### 8. Tabs
- Dynamic content
- Animation
- Active state

### 9. Toast
- Auto dismiss
- Multiple types (success, error, warning, info)
- Positioning

### 10. Avatar
- Image fallback
- Initials
- Size variants
- Status indicator

## Features

- ✅ Vue 3 Composition API
- ✅ TypeScript support
- ✅ Tailwind CSS ready
- ✅ Accessibility (ARIA)
- ✅ Dark mode support
- ✅ Fully documented
- ✅ Storybook stories included

## Installation

```bash
npm install @alphab/vue-components
```

## Usage

```vue
<template>
  <div>
    <Button variant="primary" @click="handleClick">
      Click me
    </Button>

    <Input v-model="email" type="email" placeholder="Email" />
  </div>
</template>

<script setup lang="ts">
import { Button, Input } from '@alphab/vue-components'
import { ref } from 'vue'

const email = ref('')
const handleClick = () => {
  console.log('Clicked!')
}
</script>
```

## Pricing

**Personal License:** $29
- Use in personal projects
- Unlimited projects
- Free updates

**Commercial License:** $79
- Use in commercial projects
- Unlimited projects
- Priority support
- Custom components

## Roadmap

- [ ] More components (Table, Form, Pagination)
- [ ] Nuxt 3 support
- [ ] React version
- [ ] Component generator CLI

## Support

Email: support@alphab.dev
GitHub: https://github.com/AlphaB135/vue-components

## License

MIT for buyers. Commercial redistribution prohibited.
