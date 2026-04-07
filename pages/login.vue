<template>
  <div class="login-page">
    <div class="grain"></div>

    <div class="login-box">
      <div class="brand">
        <span class="brand-mark">✦</span>
        <h1>ALIA & BRA</h1>
        <p class="brand-sub">後台管理系統</p>
      </div>

      <form class="login-form" @submit.prevent="handleLogin">
        <div class="field">
          <label>電子郵件</label>
          <input v-model="email" type="email" placeholder="admin@example.com" required />
        </div>
        <div class="field">
          <label>密碼</label>
          <input v-model="password" type="password" placeholder="••••••••" required />
        </div>

        <p v-if="errorMsg" class="error-msg">{{ errorMsg }}</p>

        <button type="submit" class="btn btn-gold login-btn" :disabled="loading">
          <span v-if="loading">登入中…</span>
          <span v-else>進入後台</span>
        </button>
      </form>
    </div>
  </div>
</template>

<script setup>
definePageMeta({ layout: false })

const router = useRouter()
const email = ref('')
const password = ref('')
const errorMsg = ref('')
const loading = ref(false)

const supabase = getSupabaseClient()

// Already logged in → redirect
const { data: { session } } = await supabase.auth.getSession()
if (session) router.push('/')

async function handleLogin() {
  loading.value = true
  errorMsg.value = ''

  const { error } = await supabase.auth.signInWithPassword({
    email: email.value,
    password: password.value,
  })

  if (error) {
    errorMsg.value = error.message
  } else {
    router.push('/')
  }
  loading.value = false
}
</script>

<style scoped>
.login-page {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--obsidian);
  position: relative;
  overflow: hidden;
}

.grain {
  position: fixed;
  inset: 0;
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noise)' opacity='0.04'/%3E%3C/svg%3E");
  pointer-events: none;
  z-index: 0;
}

/* Diagonal accent line */
.login-page::before {
  content: '';
  position: fixed;
  top: -10%;
  right: 20%;
  width: 1px;
  height: 120vh;
  background: linear-gradient(to bottom, transparent, var(--gold) 40%, var(--gold) 60%, transparent);
  opacity: 0.12;
  transform: rotate(15deg);
}

.login-box {
  position: relative;
  z-index: 1;
  width: 100%;
  max-width: 400px;
  padding: 56px 48px;
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 3px;
  animation: fadeUp 0.6s ease both;
}

@keyframes fadeUp {
  from { opacity: 0; transform: translateY(20px); }
  to   { opacity: 1; transform: translateY(0); }
}

.brand {
  text-align: center;
  margin-bottom: 48px;
}

.brand-mark {
  display: block;
  font-size: 24px;
  color: var(--gold);
  margin-bottom: 16px;
  opacity: 0.8;
}

.brand h1 {
  font-family: var(--font-display);
  font-size: 26px;
  font-weight: 400;
  letter-spacing: 0.25em;
  color: var(--ivory);
  margin-bottom: 8px;
}

.brand-sub {
  font-size: 10px;
  letter-spacing: 0.2em;
  text-transform: uppercase;
  color: var(--ivory-muted);
}

.login-form {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.login-btn {
  width: 100%;
  justify-content: center;
  padding: 14px;
  margin-top: 8px;
  font-size: 11px;
  letter-spacing: 0.18em;
}

.login-btn:disabled { opacity: 0.6; cursor: not-allowed; }

.error-msg {
  font-size: 12px;
  color: #e74c3c;
  padding: 10px 14px;
  background: var(--red-dim);
  border-radius: 2px;
  border: 1px solid rgba(192, 57, 43, 0.3);
}
</style>
