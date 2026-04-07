<template>
  <div class="admin-shell">
    <!-- Sidebar -->
    <aside class="sidebar">
      <div class="sidebar-brand">
        <span class="brand-mark">✦</span>
        <div>
          <div class="brand-name">ALIA & BRA</div>
          <div class="brand-sub">後台管理</div>
        </div>
      </div>

      <nav class="sidebar-nav">
        <NuxtLink to="/" class="nav-item" exact-active-class="active">
          <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"><rect x="3" y="3" width="7" height="7"/><rect x="14" y="3" width="7" height="7"/><rect x="3" y="14" width="7" height="7"/><rect x="14" y="14" width="7" height="7"/></svg>
          商品管理
        </NuxtLink>
        <NuxtLink to="/categories" class="nav-item" active-class="active">
          <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"><path d="M20.59 13.41l-7.17 7.17a2 2 0 0 1-2.83 0L2 12V2h10l8.59 8.59a2 2 0 0 1 0 2.82z"/><line x1="7" y1="7" x2="7.01" y2="7"/></svg>
          分類設定
        </NuxtLink>
      </nav>

      <div class="sidebar-footer">
        <button class="nav-item theme-btn" @click="toggleTheme">
          <svg v-if="isDark" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"><circle cx="12" cy="12" r="5"/><line x1="12" y1="1" x2="12" y2="3"/><line x1="12" y1="21" x2="12" y2="23"/><line x1="4.22" y1="4.22" x2="5.64" y2="5.64"/><line x1="18.36" y1="18.36" x2="19.78" y2="19.78"/><line x1="1" y1="12" x2="3" y2="12"/><line x1="21" y1="12" x2="23" y2="12"/><line x1="4.22" y1="19.78" x2="5.64" y2="18.36"/><line x1="18.36" y1="5.64" x2="19.78" y2="4.22"/></svg>
          <svg v-else width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"><path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"/></svg>
          {{ isDark ? '淺色模式' : '深色模式' }}
        </button>
        <button class="nav-item logout" @click="handleLogout">
          <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"><path d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4"/><polyline points="16 17 21 12 16 7"/><line x1="21" y1="12" x2="9" y2="12"/></svg>
          登出
        </button>
      </div>
    </aside>

    <!-- Main -->
    <main class="main-content">
      <slot />
    </main>
  </div>
</template>

<script setup>
const router = useRouter()
const supabase = getSupabaseClient()

// Auth guard
const { data: { session } } = await supabase.auth.getSession()
if (!session) router.push('/login')

async function handleLogout() {
  await supabase.auth.signOut()
  router.push('/login')
}

// Theme
const isDark = ref(true)

onMounted(() => {
  const saved = localStorage.getItem('alia-admin-theme')
  isDark.value = saved ? saved === 'dark' : false
  applyTheme(isDark.value)
})

function toggleTheme() {
  isDark.value = !isDark.value
  applyTheme(isDark.value)
  localStorage.setItem('alia-admin-theme', isDark.value ? 'dark' : 'light')
}

function applyTheme(dark) {
  if (dark) {
    document.documentElement.removeAttribute('data-theme')
  } else {
    document.documentElement.setAttribute('data-theme', 'light')
  }
}
</script>

<style scoped>
.admin-shell {
  display: flex;
  min-height: 100vh;
}

.sidebar {
  width: 220px;
  flex-shrink: 0;
  background: var(--surface);
  border-right: 1px solid var(--border);
  display: flex;
  flex-direction: column;
  position: fixed;
  top: 0;
  left: 0;
  height: 100vh;
}

.sidebar-brand {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 28px 24px;
  border-bottom: 1px solid var(--border);
}

.brand-mark {
  font-size: 18px;
  color: var(--gold);
  opacity: 0.8;
  flex-shrink: 0;
}

.brand-name {
  font-family: var(--font-display);
  font-size: 15px;
  letter-spacing: 0.15em;
  color: var(--ivory);
}

.brand-sub {
  font-size: 11px;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: var(--ivory-muted);
  margin-top: 2px;
}

.sidebar-nav {
  flex: 1;
  padding: 16px 12px;
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.nav-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px 12px;
  font-size: 14px;
  letter-spacing: 0.06em;
  color: var(--ivory-muted);
  border-radius: 2px;
  transition: all 0.2s;
  border: none;
  background: none;
  width: 100%;
  text-align: left;
  cursor: pointer;
}

.nav-item:hover { color: var(--ivory); background: var(--surface-2); }
.nav-item.active { color: var(--gold-light); background: rgba(180, 155, 110, 0.08); }

.sidebar-footer {
  padding: 12px;
  border-top: 1px solid var(--border);
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.logout { color: var(--ivory-muted); }
.logout:hover { color: #e74c3c; background: var(--red-dim); }

.theme-btn { color: var(--ivory-muted); }
.theme-btn:hover { color: var(--gold-light); background: rgba(180, 155, 110, 0.08); }

.main-content {
  flex: 1;
  margin-left: 220px;
  min-height: 100vh;
  background: var(--obsidian);
}
</style>
