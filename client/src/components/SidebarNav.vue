<template>
  <aside class="sidebar" :class="{ collapsed }">
    <div class="sidebar-brand">
      <span class="brand-mark" aria-hidden="true">
        <svg viewBox="0 0 24 24" fill="none">
          <path d="M4 16L12 4L20 16" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round"/>
          <path d="M4 20H20" stroke="currentColor" stroke-width="1.75" stroke-linecap="round"/>
        </svg>
      </span>
      <div class="brand-text" v-show="!collapsed">
        <span class="brand-name">{{ t('nav.companyName') }}</span>
        <span class="brand-subtitle">{{ t('nav.subtitle') }}</span>
      </div>
    </div>

    <nav class="sidebar-nav">
      <router-link
        v-for="item in navItems"
        :key="item.path"
        :to="item.path"
        class="nav-item"
        active-class="nav-item-active"
        :title="collapsed ? item.label : null"
      >
        <svg class="nav-icon" viewBox="0 0 20 20" fill="none">
          <path
            v-for="(d, i) in item.icon"
            :key="i"
            :d="d"
            stroke="currentColor"
            stroke-width="1.5"
            stroke-linecap="round"
            stroke-linejoin="round"
          />
        </svg>
        <span class="nav-label" v-show="!collapsed">{{ item.label }}</span>
      </router-link>
    </nav>

    <button
      type="button"
      class="sidebar-toggle"
      @click="collapsed = !collapsed"
      :title="collapsed ? 'Expand sidebar' : 'Collapse sidebar'"
    >
      <svg class="toggle-icon" :class="{ flipped: collapsed }" viewBox="0 0 20 20" fill="none">
        <path d="M12 4L6 10L12 16" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
      </svg>
    </button>
  </aside>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { useI18n } from '../composables/useI18n'

const { t } = useI18n()

const COLLAPSE_QUERY = '(max-width: 1024px)'
const mediaQuery = typeof window !== 'undefined' ? window.matchMedia(COLLAPSE_QUERY) : null

const collapsed = ref(mediaQuery ? mediaQuery.matches : false)

const handleBreakpointChange = (event) => {
  collapsed.value = event.matches
}

onMounted(() => {
  mediaQuery?.addEventListener('change', handleBreakpointChange)
})

onUnmounted(() => {
  mediaQuery?.removeEventListener('change', handleBreakpointChange)
})

// Simple inline SVG icon paths (20x20 viewBox), no icon font/library dependency.
const ICONS = {
  overview: [
    'M3 3H9V9H3V3Z',
    'M11 3H17V9H11V3Z',
    'M3 11H9V17H3V11Z',
    'M11 11H17V17H11V11Z'
  ],
  inventory: [
    'M3 6L10 2L17 6L10 10L3 6Z',
    'M3 6V14L10 18L17 14V6',
    'M10 10V18'
  ],
  orders: [
    'M6 2H14V4H6V2Z',
    'M4 4H16V18H4V4Z',
    'M7 9H13',
    'M7 12H13',
    'M7 15H10'
  ],
  finance: [
    'M2 17H18',
    'M4 17V12',
    'M8.5 17V7',
    'M13 17V10',
    'M17 17V4'
  ],
  demand: [
    'M3 14L8 9L12 13L17 6',
    'M12 6H17V11'
  ],
  reports: [
    'M5 2H12L15 5V18H5V2Z',
    'M12 2V5H15',
    'M7 9H13',
    'M7 12H13',
    'M7 15H11'
  ]
}

const navItems = computed(() => [
  { path: '/', label: t('nav.overview'), icon: ICONS.overview },
  { path: '/inventory', label: t('nav.inventory'), icon: ICONS.inventory },
  { path: '/orders', label: t('nav.orders'), icon: ICONS.orders },
  { path: '/spending', label: t('nav.finance'), icon: ICONS.finance },
  { path: '/demand', label: t('nav.demandForecast'), icon: ICONS.demand },
  { path: '/reports', label: t('nav.reports'), icon: ICONS.reports }
])
</script>

<style scoped>
.sidebar {
  position: sticky;
  top: 0;
  align-self: flex-start;
  height: 100vh;
  width: var(--sidebar-width);
  flex-shrink: 0;
  background: #ffffff;
  border-right: 1px solid #e2e8f0;
  display: flex;
  flex-direction: column;
  transition: width 0.2s ease;
  z-index: 100;
}

.sidebar.collapsed {
  width: var(--sidebar-width-collapsed);
}

.sidebar-brand {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-5) var(--space-4);
  border-bottom: 1px solid #e2e8f0;
  min-height: 64px;
}

.brand-mark {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  flex-shrink: 0;
  border-radius: var(--radius);
  background: linear-gradient(135deg, #2563eb 0%, #1e40af 100%);
  color: #ffffff;
}

.brand-mark svg {
  width: 18px;
  height: 18px;
}

.brand-text {
  display: flex;
  flex-direction: column;
  min-width: 0;
}

.brand-name {
  font-size: 0.938rem;
  font-weight: 700;
  color: #0f172a;
  letter-spacing: -0.025em;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.brand-subtitle {
  font-size: 0.75rem;
  color: #64748b;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.sidebar-nav {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: var(--space-1);
  padding: var(--space-4) var(--space-3);
  overflow-y: auto;
}

.nav-item {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-3);
  border-radius: var(--radius);
  color: #64748b;
  text-decoration: none;
  font-weight: 500;
  font-size: 0.875rem;
  border-left: 3px solid transparent;
  transition: background-color 0.15s ease, color 0.15s ease;
  white-space: nowrap;
}

.nav-item:hover {
  color: #0f172a;
  background: #f1f5f9;
}

.nav-item-active {
  color: #2563eb;
  background: #eff6ff;
  border-left-color: #2563eb;
  font-weight: 600;
}

.nav-icon {
  width: 20px;
  height: 20px;
  flex-shrink: 0;
}

.nav-label {
  overflow: hidden;
  text-overflow: ellipsis;
}

.sidebar-toggle {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: var(--space-2);
  margin: var(--space-3);
  padding: var(--space-2);
  background: none;
  border: 1px solid #e2e8f0;
  border-radius: var(--radius);
  color: #64748b;
  cursor: pointer;
  transition: background-color 0.15s ease, color 0.15s ease;
}

.sidebar-toggle:hover {
  background: #f1f5f9;
  color: #0f172a;
}

.toggle-icon {
  width: 16px;
  height: 16px;
  transition: transform 0.2s ease;
}

.toggle-icon.flipped {
  transform: rotate(180deg);
}
</style>
