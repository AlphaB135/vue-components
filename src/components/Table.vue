<template>
  <div :class="wrapperClasses">
    <div v-if="loading" class="table__loading">
      <span class="spinner"></span>
      <span>Loading...</span>
    </div>

    <div class="table__container">
      <table class="table">
        <thead>
          <tr>
            <th
              v-for="column in columns"
              :key="column.key"
              :class="thClasses(column)"
              :style="{ width: column.width }"
              @click="handleSort(column)"
            >
              <div class="table__th-content">
                <slot name="header" v-bind="{ column }">
                  {{ column.label }}
                </slot>
                <span v-if="column.sortable" class="table__sort">
                  <SortIcon v-if="sortKey !== column.key" />
                  <SortAscIcon v-else-if="sortOrder === 'asc'" />
                  <SortDescIcon v-else />
                </span>
              </div>
            </th>
          </tr>
        </thead>

        <tbody>
          <tr
            v-for="(row, index) in paginatedData"
            :key="rowKey ? row[rowKey] : index"
            :class="{
              'table__row--clickable': clickable,
              'table__row--selected': isSelected(row)
            }"
            @click="handleRowClick(row)"
          >
            <td
              v-for="column in columns"
              :key="column.key"
              :class="tdClasses(column)"
            >
              <slot
                :name="`cell-${column.key}`"
                v-bind="{ row, value: row[column.key], column, index }"
              >
                {{ formatValue(row[column.key], column) }}
              </slot>
            </td>
          </tr>

          <tr v-if="paginatedData.length === 0" class="table__empty">
            <td :colspan="columns.length">
              <slot name="empty">
                {{ emptyText }}
              </slot>
            </td>
          </tr>
        </tbody>

        <tfoot v-if="$slots.footer">
          <tr>
            <td :colspan="columns.length">
              <slot name="footer"></slot>
            </td>
          </tr>
        </tfoot>
      </table>
    </div>

    <div v-if="pagination && totalPages > 1" class="table__pagination">
      <button
        class="table__page-btn"
        :disabled="currentPage === 1"
        @click="currentPage--"
      >
        Previous
      </button>

      <span class="table__page-info">
        Page {{ currentPage }} of {{ totalPages }}
      </span>

      <button
        class="table__page-btn"
        :disabled="currentPage === totalPages"
        @click="currentPage++"
      >
        Next
      </button>

      <select v-model="pageSize" class="table__page-size">
        <option :value="10">10 per page</option>
        <option :value="25">25 per page</option>
        <option :value="50">50 per page</option>
        <option :value="100">100 per page</option>
      </select>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, ref, watch } from 'vue'

interface Column {
  key: string
  label: string
  width?: string
  sortable?: boolean
  align?: 'left' | 'center' | 'right'
  formatter?: (value: any) => string
}

interface Props {
  columns: Column[]
  data: Record<string, any>[]
  rowKey?: string
  loading?: boolean
  clickable?: boolean
  striped?: boolean
  bordered?: boolean
  hoverable?: boolean
  compact?: boolean
  pagination?: boolean
  pageSize?: number
  emptyText?: string
  selectedRows?: Record<string, any>[]
}

const props = withDefaults(defineProps<Props>(), {
  rowKey: '',
  loading: false,
  clickable: false,
  striped: false,
  bordered: false,
  hoverable: false,
  compact: false,
  pagination: false,
  pageSize: 10,
  emptyText: 'No data available',
  selectedRows: () => []
})

const emit = defineEmits<{
  'row-click': [row: Record<string, any>]
  'sort': [key: string, order: 'asc' | 'desc' | null]
  'selection-change': [rows: Record<string, any>[]]
}>()

const currentPage = ref(1)
const pageSize = ref(props.pageSize)
const sortKey = ref<string>('')
const sortOrder = ref<'asc' | 'desc' | null>(null)

const wrapperClasses = computed(() => ({
  'table__wrapper': true,
  'table--striped': props.striped,
  'table--bordered': props.bordered,
  'table--hoverable': props.hoverable,
  'table--compact': props.compact
}))

const sortedData = computed(() => {
  if (!sortKey.value || !sortOrder.value) {
    return props.data
  }

  return [...props.data].sort((a, b) => {
    const aVal = a[sortKey.value]
    const bVal = b[sortKey.value]

    if (aVal === bVal) return 0

    const comparison = aVal < bVal ? -1 : 1
    return sortOrder.value === 'asc' ? comparison : -comparison
  })
})

const totalPages = computed(() => {
  return Math.ceil(sortedData.value.length / pageSize.value)
})

const paginatedData = computed(() => {
  if (!props.pagination) {
    return sortedData.value
  }

  const start = (currentPage.value - 1) * pageSize.value
  const end = start + pageSize.value
  return sortedData.value.slice(start, end)
})

const thClasses = (column: Column) => ({
  'table__th--sortable': column.sortable,
  'table__th--sorted': sortKey.value === column.key,
  [`table--align-${column.align || 'left'}`]: true
})

const tdClasses = (column: Column) => ({
  [`table--align-${column.align || 'left'}`]: true
})

const formatValue = (value: any, column: Column) => {
  if (column.formatter) {
    return column.formatter(value)
  }
  return value
}

const handleSort = (column: Column) => {
  if (!column.sortable) return

  if (sortKey.value === column.key) {
    if (sortOrder.value === 'asc') {
      sortOrder.value = 'desc'
    } else if (sortOrder.value === 'desc') {
      sortOrder.value = null
      sortKey.value = ''
    } else {
      sortOrder.value = 'asc'
    }
  } else {
    sortKey.value = column.key
    sortOrder.value = 'asc'
  }

  emit('sort', sortKey.value, sortOrder.value)
}

const handleRowClick = (row: Record<string, any>) => {
  if (props.clickable) {
    emit('row-click', row)
  }
}

const isSelected = (row: Record<string, any>) => {
  return props.selectedRows.some(r =>
    props.rowKey ? r[props.rowKey] === row[props.rowKey] : r === row
  )
}

// Reset to page 1 when data changes
watch(() => props.data, () => {
  currentPage.value = 1
})

// Reset to page 1 when page size changes
watch(pageSize, () => {
  currentPage.value = 1
})
</script>

<script lang="ts">
const SortIcon = {
  template: `
    <svg width="14" height="14" viewBox="0 0 14 14" fill="currentColor">
      <path d="M4.5 5.5h5L7 3l-2.5 2.5zm0 3h5l-2.5 2.5-2.5-2.5z"/>
    </svg>
  `
}

const SortAscIcon = {
  template: `
    <svg width="14" height="14" viewBox="0 0 14 14" fill="currentColor">
      <path d="M4.5 6h5L7 3.5 4.5 6z"/>
    </svg>
  `
}

const SortDescIcon = {
  template: `
    <svg width="14" height="14" viewBox="0 0 14 14" fill="currentColor">
      <path d="M4.5 3.5h5L7 6 4.5 3.5z"/>
    </svg>
  `
}
</script>

<style scoped>
.table__wrapper {
  position: relative;
}

.table__loading {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  background-color: rgba(255, 255, 255, 0.8);
  z-index: 10;
  color: #6b7280;
  font-size: 0.875rem;
}

.spinner {
  display: inline-block;
  width: 1rem;
  height: 1rem;
  border: 2px solid currentColor;
  border-radius: 50%;
  border-top-color: transparent;
  animation: spin 0.6s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.table__container {
  overflow-x: auto;
  border-radius: 0.5rem;
  border: 1px solid #e5e7eb;
}

.table--bordered .table__container {
  border: 1px solid #e5e7eb;
}

.table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.875rem;
}

.table thead {
  background-color: #f9fafb;
}

.table th {
  padding: 0.75rem 1rem;
  text-align: left;
  font-weight: 600;
  color: #374151;
  border-bottom: 1px solid #e5e7eb;
  white-space: nowrap;
}

.table__th--sortable {
  cursor: pointer;
  user-select: none;
}

.table__th--sortable:hover {
  background-color: #f3f4f6;
}

.table__th-content {
  display: flex;
  align-items: center;
  gap: 0.375rem;
}

.table__sort {
  display: flex;
  color: #9ca3af;
}

.table__th--sorted .table__sort {
  color: #3b82f6;
}

.table td {
  padding: 0.75rem 1rem;
  color: #6b7280;
  border-bottom: 1px solid #f3f4f6;
}

.table--compact td,
.table--compact th {
  padding: 0.5rem 0.75rem;
}

.table--align-left {
  text-align: left;
}

.table--align-center {
  text-align: center;
}

.table--align-right {
  text-align: right;
}

.table--striped tbody tr:nth-child(even) {
  background-color: #f9fafb;
}

.table--hoverable tbody tr:hover {
  background-color: #f9fafb;
}

.table__row--clickable {
  cursor: pointer;
}

.table__row--selected {
  background-color: #eff6ff;
}

.table__empty td {
  padding: 2rem;
  text-align: center;
  color: #9ca3af;
}

.table__pagination {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0.75rem 1rem;
  border: 1px solid #e5e7eb;
  border-top: none;
  border-bottom-left-radius: 0.5rem;
  border-bottom-right-radius: 0.5rem;
  background-color: #ffffff;
}

.table__page-btn {
  padding: 0.375rem 0.75rem;
  font-size: 0.875rem;
  color: #374151;
  background-color: #ffffff;
  border: 1px solid #d1d5db;
  border-radius: 0.375rem;
  cursor: pointer;
  transition: all 0.2s ease;
}

.table__page-btn:hover:not(:disabled) {
  background-color: #f3f4f6;
}

.table__page-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.table__page-info {
  font-size: 0.875rem;
  color: #6b7280;
}

.table__page-size {
  padding: 0.375rem 0.5rem;
  font-size: 0.875rem;
  border: 1px solid #d1d5db;
  border-radius: 0.375rem;
  color: #374151;
}

/* Dark Mode */
@media (prefers-color-scheme: dark) {
  .table__loading {
    background-color: rgba(31, 41, 55, 0.8);
  }

  .table__container {
    border-color: #374151;
  }

  .table thead {
    background-color: #111827;
  }

  .table th {
    color: #e5e7eb;
    border-bottom-color: #374151;
  }

  .table__th--sortable:hover {
    background-color: #1f2937;
  }

  .table td {
    color: #9ca3af;
    border-bottom-color: #1f2937;
  }

  .table--striped tbody tr:nth-child(even),
  .table--hoverable tbody tr:hover {
    background-color: #1f2937;
  }

  .table__row--selected {
    background-color: #1e3a5f;
  }

  .table__pagination {
    background-color: #1f2937;
    border-color: #374151;
  }

  .table__page-btn {
    color: #e5e7eb;
    background-color: #1f2937;
    border-color: #4b5563;
  }

  .table__page-btn:hover:not(:disabled) {
    background-color: #374151;
  }

  .table__page-size {
    background-color: #1f2937;
    border-color: #4b5563;
    color: #e5e7eb;
  }
}
</style>
