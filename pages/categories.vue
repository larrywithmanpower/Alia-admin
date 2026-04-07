<template>
  <div class="categories-page">
    <div class="page-header">
      <div>
        <h1 class="page-title">分類設定</h1>
        <p class="page-count">共 {{ categories.length }} 個分類</p>
      </div>
    </div>

    <!-- Add form -->
    <div class="add-section">
      <form class="add-form" @submit.prevent="addCategory">
        <div class="field">
          <label>新增分類名稱</label>
          <div class="add-row">
            <input v-model="newName" type="text" placeholder="例：Swimwear" required />
            <button type="submit" class="btn btn-gold" :disabled="saving">
              {{ saving ? '新增中…' : '新增' }}
            </button>
          </div>
        </div>
        <p v-if="addError" class="form-error">{{ addError }}</p>
      </form>
    </div>

    <!-- Category list -->
    <div class="table-wrap">
      <table class="cat-table">
        <thead>
          <tr>
            <th>分類名稱</th>
            <th>商品數量</th>
            <th></th>
          </tr>
        </thead>
        <tbody>
          <tr v-if="categories.length === 0">
            <td colspan="3" class="empty-row">尚無分類</td>
          </tr>
          <tr v-for="cat in categories" :key="cat.id" class="cat-row">
            <td class="cat-name">{{ cat.name }}</td>
            <td class="cat-count">{{ productCountMap[cat.name] ?? 0 }} 件</td>
            <td class="actions-cell">
              <button
                class="action-btn danger"
                title="刪除"
                :disabled="(productCountMap[cat.name] ?? 0) > 0"
                @click="confirmDelete(cat)"
              >
                <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"><polyline points="3 6 5 6 21 6"/><path d="M19 6l-1 14a2 2 0 0 1-2 2H8a2 2 0 0 1-2-2L5 6"/><path d="M10 11v6"/><path d="M14 11v6"/><path d="M9 6V4h6v2"/></svg>
              </button>
            </td>
          </tr>
        </tbody>
      </table>
      <p class="hint">有商品的分類無法刪除，請先將商品改至其他分類。</p>
    </div>

    <!-- Delete Confirm Modal -->
    <Transition name="modal">
      <div v-if="deleteTarget" class="modal-overlay" @click.self="deleteTarget = null">
        <div class="modal modal-sm">
          <div class="modal-header">
            <h2 class="modal-title">刪除分類</h2>
            <button class="modal-close" @click="deleteTarget = null">✕</button>
          </div>
          <div class="delete-body">
            <p>確定要刪除 <strong>{{ deleteTarget.name }}</strong>？</p>
            <p class="delete-sub">此操作無法復原。</p>
          </div>
          <div class="modal-footer">
            <button class="btn btn-ghost" @click="deleteTarget = null">取消</button>
            <button class="btn btn-danger" :disabled="saving" @click="deleteCategory">
              {{ saving ? '刪除中…' : '確認刪除' }}
            </button>
          </div>
        </div>
      </div>
    </Transition>
  </div>
</template>

<script setup>
const supabase = getSupabaseClient()

const categories = ref([])
const productCountMap = ref({})
const newName = ref('')
const addError = ref('')
const saving = ref(false)
const deleteTarget = ref(null)

async function fetchAll() {
  const [{ data: cats }, { data: prods }] = await Promise.all([
    supabase.from('categories').select('*').order('sort_order').order('created_at'),
    supabase.from('products').select('category'),
  ])
  categories.value = cats || []
  const map = {}
  for (const p of prods || []) {
    map[p.category] = (map[p.category] || 0) + 1
  }
  productCountMap.value = map
}

await fetchAll()

async function addCategory() {
  saving.value = true
  addError.value = ''
  const name = newName.value.trim()
  const maxOrder = categories.value.length ? Math.max(...categories.value.map(c => c.sort_order)) : 0
  const { error } = await supabase.from('categories').insert({ name, sort_order: maxOrder + 1 })
  if (error) {
    addError.value = error.message.includes('unique') ? '此分類名稱已存在' : error.message
  } else {
    newName.value = ''
    await fetchAll()
  }
  saving.value = false
}

function confirmDelete(cat) {
  deleteTarget.value = cat
}

async function deleteCategory() {
  saving.value = true
  await supabase.from('categories').delete().eq('id', deleteTarget.value.id)
  deleteTarget.value = null
  await fetchAll()
  saving.value = false
}
</script>

<style scoped>
.categories-page {
  padding: 40px 48px;
  max-width: 800px;
}

.page-header {
  margin-bottom: 32px;
}

.page-title {
  font-family: var(--font-display);
  font-size: 36px;
  font-weight: 300;
  letter-spacing: 0.04em;
  color: var(--ivory);
  margin-bottom: 4px;
}

.page-count {
  font-size: 14px;
  color: var(--ivory-muted);
}

.add-section {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 3px;
  padding: 24px 28px;
  margin-bottom: 24px;
}

.add-row {
  display: flex;
  gap: 12px;
}

.add-row input {
  flex: 1;
  background: var(--surface-2);
  border: 1px solid var(--border);
  border-radius: 2px;
  padding: 10px 14px;
  color: var(--ivory);
  font-size: 15px;
  outline: none;
  transition: border-color 0.2s;
}

.add-row input:focus { border-color: var(--gold); }

.form-error {
  margin-top: 10px;
  font-size: 13px;
  color: #e74c3c;
  padding: 8px 12px;
  background: var(--red-dim);
  border-radius: 2px;
}

.table-wrap {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 3px;
  overflow: hidden;
}

.cat-table {
  width: 100%;
  border-collapse: collapse;
}

.cat-table th {
  font-size: 13px;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--ivory-muted);
  font-weight: 500;
  padding: 14px 20px;
  text-align: left;
  border-bottom: 1px solid var(--border);
  background: var(--surface-2);
}

.cat-table td {
  padding: 16px 20px;
  border-bottom: 1px solid rgba(180, 155, 110, 0.07);
  vertical-align: middle;
  font-size: 16px;
}

.cat-row:last-child td { border-bottom: none; }
.cat-row:hover td { background: var(--surface-2); }

.cat-name { color: var(--ivory); font-size: 16px; }
.cat-count { color: var(--ivory-muted); font-size: 14px; }

.empty-row {
  text-align: center;
  color: var(--ivory-muted);
  padding: 48px !important;
  font-size: 14px;
}

.actions-cell { text-align: right; }

.action-btn {
  background: none;
  border: 1px solid transparent;
  padding: 6px 8px;
  color: var(--ivory-muted);
  border-radius: 2px;
  transition: all 0.2s;
  display: inline-flex;
  align-items: center;
  cursor: pointer;
}
.action-btn.danger:hover { color: #e74c3c; border-color: rgba(192,57,43,0.3); background: var(--red-dim); }
.action-btn:disabled { opacity: 0.3; cursor: not-allowed; }
.action-btn:disabled:hover { color: var(--ivory-muted); border-color: transparent; background: none; }

.hint {
  font-size: 12px;
  color: var(--ivory-muted);
  padding: 12px 20px;
  border-top: 1px solid var(--border);
  background: var(--surface-2);
}

/* Modal */
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(12, 11, 10, 0.85);
  backdrop-filter: blur(4px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 100;
  padding: 24px;
}

.modal {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 3px;
  width: 100%;
  max-width: 400px;
}

.modal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 24px 28px;
  border-bottom: 1px solid var(--border);
}

.modal-title {
  font-family: var(--font-display);
  font-size: 22px;
  font-weight: 400;
  letter-spacing: 0.04em;
}

.modal-close {
  background: none;
  border: none;
  color: var(--ivory-muted);
  font-size: 14px;
  cursor: pointer;
  transition: color 0.2s;
}
.modal-close:hover { color: var(--ivory); }

.delete-body { padding: 24px 28px; }
.delete-body p { font-size: 15px; color: var(--ivory-dim); }
.delete-body strong { color: var(--ivory); }
.delete-sub { margin-top: 8px; font-size: 13px; color: var(--ivory-muted) !important; }

.modal-footer {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  padding: 20px 28px;
  border-top: 1px solid var(--border);
}

.modal-enter-active, .modal-leave-active { transition: all 0.2s ease; }
.modal-enter-from, .modal-leave-to { opacity: 0; }
</style>
