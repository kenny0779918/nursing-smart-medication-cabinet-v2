<script setup>
import { computed, ref } from 'vue'
import SystemHeader from './components/SystemHeader.vue'
import { medicines, recentRecords } from './data/medicines.js'

const page = ref('login')
const user = ref(null)
const search = ref('')
const selectedMedicine = ref(null)
const quantity = ref(1)
const records = ref([...recentRecords])
const notice = ref('')

const filteredMedicines = computed(() => {
  const keyword = search.value.trim().toLowerCase()
  if (!keyword) return medicines
  return medicines.filter(item => [item.code, item.name, item.chinese, item.cabinet].some(value => value.toLowerCase().includes(keyword)))
})

function login() {
  user.value = { name: '王小美', id: 'N02381', unit: '護理部 8A 病房' }
  page.value = 'home'
}

function goHome() {
  page.value = 'home'
  selectedMedicine.value = null
  quantity.value = 1
  search.value = ''
  notice.value = ''
}

function logout() {
  user.value = null
  page.value = 'login'
  selectedMedicine.value = null
}

function chooseMedicine(item) {
  selectedMedicine.value = item
  quantity.value = 1
  page.value = 'confirm'
}

function openCabinet() {
  if (quantity.value < 1 || quantity.value > selectedMedicine.value.stock) {
    notice.value = '取藥數量須介於 1 與目前庫存量之間。'
    return
  }
  notice.value = ''
  page.value = 'cabinet'
}

function completePickup() {
  records.value.unshift({
    time: new Date().toLocaleTimeString('zh-TW', { hour: '2-digit', minute: '2-digit', hour12: false }),
    staff: user.value.name,
    medicine: selectedMedicine.value.chinese,
    quantity: `${quantity.value} ${selectedMedicine.value.unit}`,
    status: '已完成'
  })
  page.value = 'success'
}
</script>

<template>
  <div class="app-shell">
    <SystemHeader :user="user" @home="goHome" @logout="logout" />

    <main v-if="page === 'login'" class="login-page">
      <section class="login-card">
        <div class="login-icon">▥</div>
        <p class="eyebrow">人員身分驗證</p>
        <h1>請刷讀員工識別證</h1>
        <p class="muted">將員工識別證條碼靠近掃描器，系統將自動登入。</p>
        <div class="scanner-line"><span></span></div>
        <button class="primary-button large" @click="login">模擬刷卡登入</button>
        <small>Prototype 模式：本畫面不會讀取真實員工資料</small>
      </section>
    </main>

    <main v-else class="workspace">
      <aside class="sidebar">
        <p class="nav-label">作業功能</p>
        <button :class="{ active: page === 'home' }" @click="goHome"><span>⌂</span>功能首頁</button>
        <button :class="{ active: ['medicines','confirm','cabinet','success'].includes(page) }" @click="page = 'medicines'"><span>⊞</span>取藥作業</button>
        <button disabled><span>▣</span>入庫作業<small>後續階段</small></button>
        <button disabled><span>↩</span>退藥作業<small>後續階段</small></button>
        <button disabled><span>✓</span>盤點作業<small>後續階段</small></button>
        <button disabled><span>▤</span>稽核查詢<small>後續階段</small></button>
        <div class="sidebar-footer"><b>Prototype v0.1</b><span>常備藥取藥流程</span></div>
      </aside>

      <section class="content">
        <template v-if="page === 'home'">
          <div class="page-heading"><div><p class="eyebrow">護理站作業台</p><h1>日安，{{ user.name }}</h1><p>請選擇本次要執行的藥品作業。</p></div><span class="date-chip">2026 / 09 / 01</span></div>
          <div class="metric-grid">
            <article><span class="metric-icon blue">⊞</span><div><small>今日取藥</small><strong>28</strong><span>筆</span></div></article>
            <article><span class="metric-icon teal">▦</span><div><small>藥品總庫存</small><strong>314</strong><span>件</span></div></article>
            <article><span class="metric-icon amber">!</span><div><small>低庫存提醒</small><strong>3</strong><span>項</span></div></article>
          </div>
          <h2 class="section-title">快速作業</h2>
          <div class="action-grid">
            <button class="action-card featured" @click="page = 'medicines'"><span class="big-icon">⊞</span><div><strong>常備藥取藥</strong><p>搜尋藥品、確認數量並開啟對應藥盒</p></div><b>開始作業 →</b></button>
            <button class="action-card disabled" disabled><span class="big-icon">▣</span><div><strong>藥品入庫</strong><p>後續階段開放</p></div></button>
            <button class="action-card disabled" disabled><span class="big-icon">✓</span><div><strong>盤點作業</strong><p>後續階段開放</p></div></button>
          </div>
          <h2 class="section-title">最近取藥紀錄</h2>
          <div class="table-card"><table><thead><tr><th>時間</th><th>人員</th><th>藥品</th><th>數量</th><th>狀態</th></tr></thead><tbody><tr v-for="record in records.slice(0,4)" :key="record.time + record.medicine"><td>{{ record.time }}</td><td>{{ record.staff }}</td><td>{{ record.medicine }}</td><td>{{ record.quantity }}</td><td><span class="success-badge">{{ record.status }}</span></td></tr></tbody></table></div>
        </template>

        <template v-if="page === 'medicines'">
          <div class="page-heading"><div><button class="back-link" @click="goHome">← 返回首頁</button><h1>常備藥取藥</h1><p>搜尋並選擇本次要取出的藥品。</p></div><div class="stepper"><b class="current">1</b><span></span><b>2</b><span></span><b>3</b></div></div>
          <div class="search-card"><label>搜尋藥品</label><div class="search-box"><span>⌕</span><input v-model="search" placeholder="輸入藥品代碼、中文名稱、學名或藥盒編號"/><button v-if="search" @click="search = ''">清除</button></div><small>找到 {{ filteredMedicines.length }} 項藥品</small></div>
          <div class="medicine-grid">
            <article v-for="item in filteredMedicines" :key="item.id" class="medicine-card">
              <div class="medicine-top"><span class="category-badge">{{ item.category }}</span><span class="stock" :class="{ low: item.stock < 15 }">庫存 {{ item.stock }} {{ item.unit }}</span></div>
              <h3>{{ item.chinese }}</h3><p>{{ item.name }}</p><dl><div><dt>藥品代碼</dt><dd>{{ item.code }}</dd></div><div><dt>藥盒位置</dt><dd>{{ item.cabinet }}</dd></div></dl>
              <button class="outline-button" @click="chooseMedicine(item)">選擇此藥品</button>
            </article>
          </div>
        </template>

        <template v-if="page === 'confirm'">
          <div class="page-heading"><div><button class="back-link" @click="page = 'medicines'">← 重新選擇藥品</button><h1>確認取藥內容</h1><p>確認藥品與數量後，系統將開啟對應藥盒。</p></div><div class="stepper"><b class="done">✓</b><span class="done-line"></span><b class="current">2</b><span></span><b>3</b></div></div>
          <div class="confirm-layout">
            <article class="summary-card"><p class="eyebrow">本次取藥</p><h2>{{ selectedMedicine.chinese }}</h2><p class="english-name">{{ selectedMedicine.name }}</p><div class="summary-info"><div><small>藥品代碼</small><strong>{{ selectedMedicine.code }}</strong></div><div><small>藥盒位置</small><strong>{{ selectedMedicine.cabinet }}</strong></div><div><small>目前庫存</small><strong>{{ selectedMedicine.stock }} {{ selectedMedicine.unit }}</strong></div></div></article>
            <article class="quantity-card"><label for="quantity">取藥數量</label><div class="quantity-control"><button @click="quantity = Math.max(1, quantity - 1)">−</button><input id="quantity" v-model.number="quantity" type="number" min="1" :max="selectedMedicine.stock"/><button @click="quantity = Math.min(selectedMedicine.stock, quantity + 1)">＋</button><span>{{ selectedMedicine.unit }}</span></div><p v-if="notice" class="error-message">{{ notice }}</p><div class="notice-box">確認後將開啟 <strong>{{ selectedMedicine.cabinet }}</strong> 號藥盒，請留意藥盒開啟狀態。</div><button class="primary-button full" @click="openCabinet">確認並開啟藥盒</button></article>
          </div>
        </template>

        <template v-if="page === 'cabinet'">
          <div class="focus-panel">
            <div class="pulse-icon">↗</div><p class="eyebrow">藥盒已開啟</p><h1>請從藥櫃 {{ selectedMedicine.cabinet }} 號<br>取出藥品</h1>
            <div class="pickup-detail"><div><small>藥品</small><strong>{{ selectedMedicine.chinese }}</strong></div><div><small>數量</small><strong>{{ quantity }} {{ selectedMedicine.unit }}</strong></div></div>
            <div class="warning-bar">取出後請確實關閉藥盒，再按下方按鈕完成作業。</div><button class="primary-button large" @click="completePickup">模擬藥盒已關閉</button>
          </div>
        </template>

        <template v-if="page === 'success'">
          <div class="focus-panel success-panel">
            <div class="success-check">✓</div><p class="eyebrow">作業完成</p><h1>取藥紀錄已建立</h1><p>本次取藥已完成，庫存異動將於正式系統中同步保存。</p>
            <div class="receipt"><div><span>作業人員</span><strong>{{ user.name }}（{{ user.id }}）</strong></div><div><span>藥品</span><strong>{{ selectedMedicine.chinese }}</strong></div><div><span>取藥數量</span><strong>{{ quantity }} {{ selectedMedicine.unit }}</strong></div><div><span>藥盒位置</span><strong>{{ selectedMedicine.cabinet }}</strong></div></div>
            <div class="button-row"><button class="outline-button" @click="goHome">返回首頁</button><button class="primary-button" @click="page = 'medicines'; selectedMedicine = null; quantity = 1">繼續取藥</button></div>
          </div>
        </template>
      </section>
    </main>
  </div>
</template>
