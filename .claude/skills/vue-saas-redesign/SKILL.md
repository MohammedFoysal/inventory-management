---
name: vue-saas-redesign
description: Guidelines for redesigning a Vue 3 application's UI into a modern SaaS-style interface with a vertical left sidebar nav (replacing a top nav bar), consistent spacing, and a polished professional look. Use this skill when asked to redesign, modernize, or restyle the client UI, or to convert a top navbar layout into a sidebar layout.
---

# Vue 3 SaaS Redesign Guidelines

This skill provides a repeatable approach for converting a Vue 3 app's top-nav layout
into a modern SaaS-style shell: a fixed vertical sidebar on the left, a content area
with consistent spacing, and refined visual polish. Follow this skill any time the
task is "redesign the UI", "modernize the layout", "add a sidebar nav", or similar.

**This skill produces `.vue` file changes. Per the project's root `CLAUDE.md`, ANY
creation or significant modification of a `.vue` file MUST be delegated to the
`vue-expert` subagent — do not write template/script/style changes directly.**

## When to Use

- Replacing a top navigation bar with a left vertical sidebar
- General "make this look more professional / modern / SaaS-like" requests
- Introducing a consistent spacing/layout system across views
- Polishing an existing layout without changing its underlying data or routes

## Goal

Transform the app shell into this structure, without changing routes, API calls, or
business logic — this is a presentation-layer redesign:

```
┌───────────┬─────────────────────────────────────┐
│           │  Page header (title, filters, etc.)  │
│  Sidebar  ├─────────────────────────────────────┤
│  (fixed)  │                                       │
│           │        Page content                  │
│  - nav    │        (views/*.vue, unchanged)       │
│  - brand  │                                       │
│           │                                       │
└───────────┴─────────────────────────────────────┘
```

## 1. Layout Architecture

Restructure the app shell (typically `App.vue`) into a flex/grid container with two
regions: a fixed-width sidebar and a fluid content area. Keep `<router-view>` as the
only thing rendering page content — do not touch individual views' internal markup
unless spacing needs adjusting.

```vue
<template>
  <div class="app-shell">
    <SidebarNav />
    <div class="app-main">
      <router-view />
    </div>
  </div>
</template>

<style scoped>
.app-shell {
  display: flex;
  min-height: 100vh;
}

.app-main {
  flex: 1;
  margin-left: 240px; /* matches sidebar width */
  padding: 32px;
  background: #f8fafc;
  min-width: 0; /* prevents flex overflow with wide tables/charts */
}
</style>
```

## 2. Sidebar Component Pattern

Create a dedicated `SidebarNav.vue` component (in `client/src/components/`). Use
`vue-router`'s `<router-link>` with `active-class` for automatic active-state
highlighting — don't manually track the current route.

```vue
<template>
  <aside class="sidebar" :class="{ collapsed }">
    <div class="sidebar-brand">
      <span class="brand-mark">FI</span>
      <span class="brand-name" v-show="!collapsed">Factory Inventory</span>
    </div>

    <nav class="sidebar-nav">
      <router-link
        v-for="item in navItems"
        :key="item.path"
        :to="item.path"
        class="nav-item"
        active-class="nav-item-active"
      >
        <component :is="item.icon" class="nav-icon" />
        <span v-show="!collapsed">{{ item.label }}</span>
      </router-link>
    </nav>

    <button class="sidebar-toggle" @click="collapsed = !collapsed">
      {{ collapsed ? '»' : '«' }}
    </button>
  </aside>
</template>

<script>
import { ref } from 'vue'

export default {
  name: 'SidebarNav',
  setup() {
    const collapsed = ref(false)

    const navItems = [
      { path: '/', label: 'Dashboard', icon: 'IconDashboard' },
      { path: '/inventory', label: 'Inventory', icon: 'IconInventory' },
      { path: '/orders', label: 'Orders', icon: 'IconOrders' },
      { path: '/demand', label: 'Demand', icon: 'IconDemand' },
      { path: '/spending', label: 'Spending', icon: 'IconSpending' },
      { path: '/reports', label: 'Reports', icon: 'IconReports' }
    ]

    return { collapsed, navItems }
  }
}
</script>
```

**Notes:**
- Keep the nav item list as plain data (path/label/icon), not hardcoded markup per link — easier to extend.
- Use simple inline SVG icon components, not an icon font/library dependency, to match the project's existing "custom SVG" convention.
- Sidebar width should be a single source of truth (CSS variable) shared with `.app-main`'s margin.

## 3. Spacing System

Define a consistent spacing scale and apply it everywhere instead of ad-hoc pixel
values. Add these as CSS custom properties on `:root` (in `App.vue` or a shared
styles file):

```css
:root {
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-5: 24px;
  --space-6: 32px;
  --space-7: 48px;

  --sidebar-width: 240px;
  --sidebar-width-collapsed: 72px;
  --radius: 10px;
}
```

Apply consistently:
- Page-level padding: `--space-6` (32px)
- Card/panel padding: `--space-5` (24px)
- Gap between cards in a grid: `--space-4` (16px)
- Gap between label and value, icon and text: `--space-2` (8px)

Avoid mixing arbitrary values (e.g. `13px`, `27px`) — snap everything to the scale.

## 4. Visual Polish Checklist

Follow the project's existing design system (`CLAUDE.md`): slate/gray palette
(`#0f172a` ink, `#64748b` muted text, `#e2e8f0` borders), status colors for
green/blue/yellow/red, no emojis anywhere in the UI.

- [ ] Consistent border radius across cards, buttons, inputs (`--radius`, e.g. 8-10px)
- [ ] Subtle shadows only on elevated elements (modals, dropdowns) — flat cards use a 1px border instead
- [ ] One consistent card style reused everywhere (border + radius + padding), not per-view variants
- [ ] Clear typographic hierarchy: page title > section heading > card label > body text, with no more than 4 font sizes
- [ ] Active/hover states on all interactive elements (nav items, buttons, table rows)
- [ ] Icons sized consistently (e.g. 18-20px) and aligned to text baseline
- [ ] Sidebar active route is visually obvious (background tint + left accent bar or bold text)
- [ ] Empty/loading/error states styled, not just plain text

## 5. Migrating Existing Views

- Remove the old top nav bar component/markup once the sidebar replaces it — don't leave both mounted.
- Check each view (`client/src/views/*.vue`) for hardcoded top-nav-relative spacing (e.g. `margin-top` offsets that assumed a top bar) and adjust for the new shell padding instead.
- Leave view-internal logic (data loading, computed properties, filters) untouched — this is a layout/styling change only.
- If the top nav previously held global elements (profile menu, language switcher, filters), relocate them into the sidebar footer/header or into the content area's page header — don't drop functionality silently.

## 6. Responsive Behavior

- Above ~1024px: full sidebar with labels, as designed.
- Below ~1024px: collapse to icon-only sidebar (use the `collapsed` state), or hide behind a toggle button in the content header.
- Never let the sidebar overlap content or cause horizontal scroll — verify at common widths (1440px, 1024px, 768px).

## 7. Testing the Redesign

Per root `CLAUDE.md`, use Playwright MCP tools against `http://localhost:3000` to
visually verify:

1. Sidebar renders on every route and highlights the active link correctly.
2. Navigating between views doesn't remount/flicker the sidebar.
3. Content area spacing looks consistent across at least 3 different views.
4. Collapse/expand toggle (if implemented) works and persists layout without breaking charts/tables.
5. No layout regressions at mobile widths.

## Key Reminders

- **Delegate all `.vue` work to `vue-expert`** — this is a mandatory project rule, not optional.
- **Composition API only** — no Options API, per `client/CLAUDE.md`.
- **No emojis** in nav labels, icons, or any UI text.
- **Single source of truth for sidebar width** — a CSS variable, referenced by both the sidebar and the content area margin.
- **Don't touch business logic** — this skill is scoped to layout and visual styling, not data flow or API behavior.
- **Verify in the browser** with Playwright before calling the redesign done — a skill/agent producing template diffs is not the same as a confirmed working UI.
