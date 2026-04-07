<template>
  <div class="products-page">
    <!-- Header -->
    <div class="page-header">
      <div>
        <h1 class="page-title">Products</h1>
        <p class="page-count">共 {{ products.length }} 件商品</p>
      </div>
      <button class="btn btn-gold" @click="openNew">
        <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><line x1="12" y1="5" x2="12" y2="19"/><line x1="5" y1="12" x2="19" y2="12"/></svg>
        新增商品
      </button>
    </div>

    <!-- Filters -->
    <div class="filters-bar">
      <div class="search-wrap">
        <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"><circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/></svg>
        <input v-model="search" type="text" placeholder="搜尋商品…" class="search-input" />
      </div>
      <div class="filter-tabs">
        <button
          v-for="f in categories"
          :key="f"
          class="filter-tab"
          :class="{ active: activeCategory === f }"
          @click="activeCategory = f"
        >{{ f }}</button>
      </div>
    </div>

    <!-- Table -->
    <div class="table-wrap">
      <table class="product-table">
        <thead>
          <tr>
            <th>商品</th>
            <th>分類</th>
            <th>價格</th>
            <th>標籤</th>
            <th>首頁精選</th>
            <th>購買連結</th>
            <th></th>
          </tr>
        </thead>
        <tbody>
          <tr v-if="filteredProducts.length === 0">
            <td colspan="7" class="empty-row">找不到商品</td>
          </tr>
          <tr v-for="p in filteredProducts" :key="p.id" class="product-row">
            <td class="product-name-cell">
              <div class="product-thumb" :style="p.image_url ? `background-image: url(${p.image_url})` : ''">
                <span v-if="!p.image_url">✦</span>
              </div>
              <span class="product-name">{{ p.name }}</span>
            </td>
            <td><span class="category-tag">{{ p.category }}</span></td>
            <td class="price-cell">${{ p.price }}</td>
            <td>
              <span v-if="p.badge" class="badge-tag">{{ p.badge }}</span>
              <span v-else class="no-value">—</span>
            </td>
            <td>
              <span class="featured-dot" :class="{ on: p.featured }"></span>
            </td>
            <td>
              <span v-if="p.shop_url" class="link-indicator">已設定</span>
              <span v-else class="no-value">—</span>
            </td>
            <td class="actions-cell">
              <button class="action-btn" title="編輯" @click="openEdit(p)">
                <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"><path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"/><path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"/></svg>
              </button>
              <button class="action-btn danger" title="刪除" @click="confirmDelete(p)">
                <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"><polyline points="3 6 5 6 21 6"/><path d="M19 6l-1 14a2 2 0 0 1-2 2H8a2 2 0 0 1-2-2L5 6"/><path d="M10 11v6"/><path d="M14 11v6"/><path d="M9 6V4h6v2"/></svg>
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Product Modal -->
    <Transition name="modal">
      <div v-if="showModal" class="modal-overlay" @click.self="closeModal">
        <div class="modal">
          <div class="modal-header">
            <h2 class="modal-title">{{ editingProduct ? '編輯商品' : '新增商品' }}</h2>
            <button class="modal-close" @click="closeModal">✕</button>
          </div>

          <form class="modal-form" @submit.prevent="saveProduct">
            <div class="form-grid">
              <div class="field">
                <label>商品名稱 *</label>
                <input v-model="form.name" type="text" placeholder="例：Delicieux Lace Bralette" required />
              </div>

              <div class="field">
                <label>分類 *</label>
                <select v-model="form.category" required>
                  <option value="">選擇分類</option>
                  <option v-for="c in productCategories" :key="c" :value="c">{{ c }}</option>
                </select>
              </div>

              <div class="field">
                <label>價格（NT$）*</label>
                <input v-model="form.price" type="number" min="0" step="0.01" placeholder="980" required />
              </div>

              <div class="field">
                <label>標籤</label>
                <input v-model="form.badge" type="text" placeholder="例：NEW、BESTSELLER、LIMITED" />
              </div>

              <div class="field span-2">
                <label>商品圖片</label>
                <div class="image-tabs">
                  <button type="button" class="image-tab" :class="{ active: imageMode === 'url' }" @click="imageMode = 'url'">貼上網址</button>
                  <button type="button" class="image-tab" :class="{ active: imageMode === 'upload' }" @click="imageMode = 'upload'">上傳圖片</button>
                </div>
                <input v-if="imageMode === 'url'" v-model="form.image_url" type="url" placeholder="https://…" />
                <div v-else class="upload-area">
                  <input ref="fileInput" type="file" accept="image/*" class="file-input" @change="handleFileChange" />
                  <div class="upload-box"
                    @click="fileInput.value.click()"
                    @dragover.prevent="isDragging = true"
                    @dragleave="isDragging = false"
                    @drop.prevent="handleDrop"
                    :class="{ dragging: isDragging }"
                  >
                    <template v-if="uploadPreview">
                      <img :src="uploadPreview" class="upload-preview" />
                    </template>
                    <template v-else>
                      <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="17 8 12 3 7 8"/><line x1="12" y1="3" x2="12" y2="15"/></svg>
                      <span>點擊選擇圖片</span>
                    </template>
                  </div>
                  <p v-if="uploadingImage" class="upload-status">上傳中…</p>
                </div>
                <p v-if="form.image_url && imageMode === 'url'" class="image-hint">目前圖片：<a :href="form.image_url" target="_blank">預覽</a></p>
              </div>

              <div class="field span-2">
                <label>購買連結（蝦皮 / momo）</label>
                <input v-model="form.shop_url" type="url" placeholder="https://shopee.tw/…" />
              </div>

              <div class="field span-2">
                <div class="toggle-wrap">
                  <label class="toggle">
                    <input v-model="form.featured" type="checkbox" />
                    <span class="toggle-slider"></span>
                  </label>
                  <span class="toggle-label">顯示在首頁精選區</span>
                </div>
              </div>
            </div>

            <p v-if="formError" class="form-error">{{ formError }}</p>

            <div class="modal-footer">
              <button type="button" class="btn btn-ghost" @click="closeModal">取消</button>
              <button type="submit" class="btn btn-gold" :disabled="saving">
                {{ saving ? '儲存中…' : (editingProduct ? '儲存變更' : '建立商品') }}
              </button>
            </div>
          </form>
        </div>
      </div>
    </Transition>

    <!-- Delete Confirm Modal -->
    <Transition name="modal">
      <div v-if="deleteTarget" class="modal-overlay" @click.self="deleteTarget = null">
        <div class="modal modal-sm">
          <div class="modal-header">
            <h2 class="modal-title">刪除商品</h2>
            <button class="modal-close" @click="deleteTarget = null">✕</button>
          </div>
          <div class="delete-body">
            <p>確定要刪除 <strong>{{ deleteTarget.name }}</strong>？</p>
            <p class="delete-sub">此操作無法復原。</p>
          </div>
          <div class="modal-footer">
            <button class="btn btn-ghost" @click="deleteTarget = null">取消</button>
            <button class="btn btn-danger" :disabled="saving" @click="deleteProduct">
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

const categoryNames = ref([])
const categories = computed(() => ['全部', ...categoryNames.value])
const productCategories = computed(() => categoryNames.value)

const products = ref([])
const search = ref('')
const activeCategory = ref('全部')
const showModal = ref(false)
const editingProduct = ref(null)
const deleteTarget = ref(null)
const saving = ref(false)
const formError = ref('')
const imageMode = ref('url')
const fileInput = ref(null)
const uploadPreview = ref(null)
const uploadingImage = ref(false)
const isDragging = ref(false)

const defaultForm = () => ({
  name: '',
  category: '',
  price: '',
  badge: '',
  image_url: '',
  shop_url: '',
  featured: false,
})

const form = ref(defaultForm())

const filteredProducts = computed(() => {
  let list = products.value
  if (activeCategory.value !== '全部') list = list.filter(p => p.category === activeCategory.value)
  if (search.value) {
    const q = search.value.toLowerCase()
    list = list.filter(p => p.name.toLowerCase().includes(q))
  }
  return list
})

async function fetchProducts() {
  const { data } = await supabase.from('products').select('*').order('created_at', { ascending: false })
  products.value = data || []
}

const { data: catData } = await supabase.from('categories').select('name').order('sort_order').order('created_at')
categoryNames.value = (catData || []).map(c => c.name)

await fetchProducts()

function resetImageState() {
  imageMode.value = 'url'
  uploadPreview.value = null
  uploadingImage.value = false
}

function openNew() {
  editingProduct.value = null
  form.value = defaultForm()
  formError.value = ''
  resetImageState()
  showModal.value = true
}

function openEdit(p) {
  editingProduct.value = p
  form.value = {
    name: p.name,
    category: p.category,
    price: p.price,
    badge: p.badge || '',
    image_url: p.image_url || '',
    shop_url: p.shop_url || '',
    featured: p.featured,
  }
  formError.value = ''
  resetImageState()
  showModal.value = true
}

function closeModal() {
  showModal.value = false
  editingProduct.value = null
}

function compressImage(file, maxWidth = 1200, quality = 0.82) {
  return new Promise((resolve) => {
    const img = new Image()
    img.onload = () => {
      const scale = Math.min(1, maxWidth / img.width)
      const canvas = document.createElement('canvas')
      canvas.width = img.width * scale
      canvas.height = img.height * scale
      canvas.getContext('2d').drawImage(img, 0, 0, canvas.width, canvas.height)
      canvas.toBlob(resolve, 'image/jpeg', quality)
    }
    img.src = URL.createObjectURL(file)
  })
}

async function handleDrop(e) {
  isDragging.value = false
  const file = e.dataTransfer.files[0]
  if (!file || !file.type.startsWith('image/')) return
  await uploadFile(file)
}

async function handleFileChange(e) {
  const file = e.target.files[0]
  if (!file) return
  await uploadFile(file)
}

async function uploadFile(file) {
  uploadPreview.value = URL.createObjectURL(file)
  uploadingImage.value = true
  const compressed = await compressImage(file)
  const filename = `products/${Date.now()}.jpg`
  const { error } = await supabase.storage.from('website-assets').upload(filename, compressed, { contentType: 'image/jpeg', upsert: true })
  if (!error) {
    const { data: urlData } = supabase.storage.from('website-assets').getPublicUrl(filename)
    form.value.image_url = urlData.publicUrl
  }
  uploadingImage.value = false
}


async function saveProduct() {
  saving.value = true
  formError.value = ''

  const payload = {
    name: form.value.name,
    category: form.value.category,
    price: parseFloat(form.value.price),
    badge: form.value.badge || null,
    image_url: form.value.image_url || null,
    shop_url: form.value.shop_url || null,
    featured: form.value.featured,
  }

  let error
  if (editingProduct.value) {
    // 若圖片換了，刪除舊圖
    const oldUrl = editingProduct.value.image_url
    const newUrl = form.value.image_url
    if (oldUrl && newUrl !== oldUrl && oldUrl.includes('website-assets')) {
      const path = oldUrl.split('/website-assets/')[1]
      if (path) await supabase.storage.from('website-assets').remove([path])
    }
    ;({ error } = await supabase.from('products').update(payload).eq('id', editingProduct.value.id))
  } else {
    ;({ error } = await supabase.from('products').insert(payload))
  }

  if (error) {
    formError.value = error.message
  } else {
    await fetchProducts()
    closeModal()
  }
  saving.value = false
}

function confirmDelete(p) {
  deleteTarget.value = p
}

async function deleteProduct() {
  saving.value = true
  const { error } = await supabase.from('products').delete().eq('id', deleteTarget.value.id)
  if (!error) {
    await fetchProducts()
    deleteTarget.value = null
  }
  saving.value = false
}
</script>

<style scoped>
.products-page {
  padding: 40px 48px;
  max-width: 1200px;
}

.page-header {
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
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
  letter-spacing: 0.05em;
}

/* Filters */
.filters-bar {
  display: flex;
  align-items: center;
  gap: 16px;
  margin-bottom: 24px;
  flex-wrap: wrap;
}

.search-wrap {
  display: flex;
  align-items: center;
  gap: 10px;
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 2px;
  padding: 8px 14px;
  flex: 1;
  max-width: 280px;
}

.search-wrap svg { color: var(--ivory-muted); flex-shrink: 0; }

.search-input {
  background: none;
  border: none;
  outline: none;
  color: var(--ivory);
  font-size: 13px;
  width: 100%;
}
.search-input::placeholder { color: var(--ivory-muted); }

.filter-tabs {
  display: flex;
  gap: 4px;
}

.filter-tab {
  background: none;
  border: 1px solid transparent;
  padding: 7px 16px;
  font-size: 13px;
  letter-spacing: 0.08em;
  color: var(--ivory-muted);
  border-radius: 2px;
  transition: all 0.2s;
}
.filter-tab:hover { color: var(--ivory); border-color: var(--border); }
.filter-tab.active { color: var(--gold-light); border-color: var(--gold); background: rgba(180,155,110,0.07); }

/* Table */
.table-wrap {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 3px;
  overflow: hidden;
}

.product-table {
  width: 100%;
  border-collapse: collapse;
}

.product-table th {
  font-size: 13px;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--ivory-muted);
  font-weight: 500;
  padding: 14px 16px;
  text-align: left;
  border-bottom: 1px solid var(--border);
  background: var(--surface-2);
}

.product-table td {
  font-size: 15px;
  padding: 16px 16px;
  border-bottom: 1px solid rgba(180, 155, 110, 0.07);
  vertical-align: middle;
}

.product-row:last-child td { border-bottom: none; }

.product-row:hover td { background: var(--surface-2); }

.empty-row {
  text-align: center;
  color: var(--ivory-muted);
  padding: 48px !important;
  font-size: 13px;
}

.product-name-cell {
  display: flex;
  align-items: center;
  gap: 12px;
}

.product-thumb {
  width: 40px;
  height: 50px;
  background: var(--surface-3);
  border: 1px solid var(--border);
  border-radius: 2px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  background-size: cover;
  background-position: center;
  color: var(--gold);
  font-size: 12px;
  opacity: 0.5;
}

.product-name {
  font-size: 15px;
  color: var(--ivory);
}

.category-tag {
  font-size: 11px;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: var(--ivory-muted);
  background: var(--surface-3);
  padding: 3px 8px;
  border-radius: 2px;
}

.price-cell {
  font-size: 15px;
  color: var(--gold-light);
  font-weight: 500;
}

.badge-tag {
  font-size: 9px;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--gold);
  border: 1px solid rgba(180, 155, 110, 0.4);
  padding: 2px 7px;
  border-radius: 2px;
}

.no-value { color: var(--ivory-muted); }

.featured-dot {
  display: inline-block;
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: var(--surface-3);
  border: 1px solid var(--border);
}
.featured-dot.on { background: var(--gold); border-color: var(--gold); box-shadow: 0 0 6px rgba(180,155,110,0.4); }

.link-indicator {
  font-size: 10px;
  color: var(--green);
  background: var(--green-dim);
  padding: 3px 8px;
  border-radius: 2px;
  letter-spacing: 0.05em;
}

.actions-cell {
  text-align: right;
  white-space: nowrap;
}

.action-btn {
  background: none;
  border: 1px solid transparent;
  padding: 6px 8px;
  color: var(--ivory-muted);
  border-radius: 2px;
  transition: all 0.2s;
  display: flex;
  align-items: center;
}
.action-btn:hover { color: var(--ivory); border-color: var(--border); background: var(--surface-2); }
.action-btn.danger:hover { color: #e74c3c; border-color: rgba(192, 57, 43, 0.3); background: var(--red-dim); }

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
  max-width: 560px;
  max-height: 90vh;
  overflow-y: auto;
}

.modal-sm { max-width: 400px; }

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
  padding: 4px;
  transition: color 0.2s;
}
.modal-close:hover { color: var(--ivory); }

.modal-form { padding: 28px; }

.form-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
  margin-bottom: 20px;
}

.image-tabs {
  display: flex;
  gap: 4px;
  margin-bottom: 10px;
}

.image-tab {
  background: none;
  border: 1px solid var(--border);
  padding: 6px 14px;
  font-size: 12px;
  letter-spacing: 0.06em;
  color: var(--ivory-muted);
  border-radius: 2px;
  cursor: pointer;
  transition: all 0.2s;
  font-family: var(--font-ui);
}
.image-tab.active { color: var(--gold-light); border-color: var(--gold); background: rgba(180,155,110,0.07); }
.image-tab:hover:not(.active) { color: var(--ivory); }

.upload-area { display: flex; flex-direction: column; gap: 8px; }

.file-input { display: none; }

.upload-box {
  border: 1px dashed var(--border);
  border-radius: 2px;
  height: 120px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 8px;
  cursor: pointer;
  color: var(--ivory-muted);
  font-size: 13px;
  transition: border-color 0.2s;
  overflow: hidden;
}
.upload-box:hover { border-color: var(--gold); color: var(--ivory); }
.upload-box.dragging { border-color: var(--gold); background: rgba(180,155,110,0.07); color: var(--ivory); }

.upload-preview {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.upload-status { font-size: 12px; color: var(--ivory-muted); }

.image-hint { font-size: 12px; color: var(--ivory-muted); margin-top: 6px; }
.image-hint a { color: var(--gold); }

.span-2 { grid-column: 1 / -1; }

.form-error {
  font-size: 12px;
  color: #e74c3c;
  padding: 10px 14px;
  background: var(--red-dim);
  border-radius: 2px;
  border: 1px solid rgba(192, 57, 43, 0.3);
  margin-bottom: 16px;
}

.modal-footer {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  padding: 20px 28px;
  border-top: 1px solid var(--border);
}

.modal-form .modal-footer {
  padding: 0;
  border-top: none;
  margin-top: 8px;
}

.delete-body {
  padding: 24px 28px;
}
.delete-body p { font-size: 14px; color: var(--ivory-dim); line-height: 1.6; }
.delete-body strong { color: var(--ivory); }
.delete-sub { margin-top: 8px; font-size: 12px; color: var(--ivory-muted) !important; }

/* Transitions */
.modal-enter-active, .modal-leave-active { transition: all 0.2s ease; }
.modal-enter-from, .modal-leave-to { opacity: 0; }
.modal-enter-from .modal, .modal-leave-to .modal { transform: scale(0.96) translateY(8px); }
</style>
