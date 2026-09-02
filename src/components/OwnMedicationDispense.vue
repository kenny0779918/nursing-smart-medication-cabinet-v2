<script setup>
import { computed, ref } from 'vue'

defineProps({ operator: Object })
const emit = defineEmits(['back', 'busy'])

const view = ref('list')
const query = ref({ ward:'全部', room:'全部', bed:'', patient:'', medicine:'', status:'可取藥' })
const selected = ref(null)
const dispenseQty = ref(1)
const validationError = ref('')
const drawerState = ref('closed')
const records = ref([
  { id:'OWN-20251018-001', bed:'A103-2', patientId:'0001234567', patient:'王小明', ward:'一般病房', room:'A103', code:'XARELTO-20', medicine:'拜瑞妥膜衣錠 20mg', generic:'Xarelto 20mg/tab', received:'2025-10-18', expiry:'2026-08-31', stock:28, unit:'顆', drawer:'21', status:'可取藥' },
  { id:'OWN-20251017-002', bed:'A105-1', patientId:'0002345678', patient:'李大華', ward:'一般病房', room:'A105', code:'SINGULAIR-10', medicine:'欣流膜衣錠 10mg', generic:'Singulair 10mg/tab', received:'2025-10-17', expiry:'2026-07-15', stock:14, unit:'顆', drawer:'22', status:'可取藥' },
  { id:'OWN-20251016-003', bed:'B201-1', patientId:'0003456789', patient:'張美玲', ward:'內科病房', room:'B201', code:'SYMBICORT-160', medicine:'吸必擴都保定量粉狀吸入劑', generic:'Symbicort 160/4.5mcg/inhaler', received:'2025-10-16', expiry:'2026-05-30', stock:1, unit:'支', drawer:'23', status:'可取藥' },
  { id:'OWN-20251016-004', bed:'B202-2', patientId:'0004567890', patient:'陳建宏', ward:'內科病房', room:'B202', code:'LIPITOR-20', medicine:'立普妥膜衣錠 20mg', generic:'Lipitor 20mg/tab', received:'2025-10-16', expiry:'2026-09-10', stock:20, unit:'顆', drawer:'24', status:'可取藥' },
  { id:'OWN-20251015-005', bed:'C301-1', patientId:'0005678901', patient:'林志偉', ward:'外科病房', room:'C301', code:'ELIQUIS-5', medicine:'艾必克凝膜衣錠 5mg', generic:'Eliquis 5mg/tab', received:'2025-10-15', expiry:'2026-06-20', stock:18, unit:'顆', drawer:'25', status:'可取藥' },
  { id:'OWN-20251014-006', bed:'C302-3', patientId:'0006789012', patient:'黃淑芬', ward:'外科病房', room:'C302', code:'TRELEGY', medicine:'全再樂易利達吸入劑', generic:'Trelegy Ellipta / inhaler', received:'2025-10-14', expiry:'2026-04-25', stock:1, unit:'支', drawer:'26', status:'可取藥' }
])

const filteredRecords = computed(() => records.value.filter(item =>
  (query.value.ward==='全部'||item.ward===query.value.ward) &&
  (query.value.room==='全部'||item.room===query.value.room) &&
  (!query.value.bed||item.bed.includes(query.value.bed.trim())) &&
  (!query.value.patient||item.patient.includes(query.value.patient.trim())) &&
  (!query.value.medicine||`${item.medicine} ${item.generic}`.toLowerCase().includes(query.value.medicine.trim().toLowerCase())) &&
  (query.value.status==='全部'||item.status===query.value.status)
))

function chooseRecord(record){ selected.value=record; dispenseQty.value=1; validationError.value=''; view.value='confirm' }
function validateQuantity(){
  const qty=Number(dispenseQty.value)
  if(!Number.isInteger(qty)||qty<=0){ validationError.value='請輸入大於 0 的整數取藥數量。'; return false }
  if(qty>selected.value.stock){ validationError.value=`取藥數量不可超過目前庫存 ${selected.value.stock} ${selected.value.unit}。`; return false }
  validationError.value=''; return true
}
function startDispensing(){
  if(!validateQuantity()) return
  if(selected.value.status!=='可取藥'||selected.value.stock<Number(dispenseQty.value)){ validationError.value='資料已異動，請返回清單重新整理後再操作。'; return }
  drawerState.value='open'; view.value='dispensing'; emit('busy',true)
}
function closeDrawer(){
  drawerState.value='closed'
  selected.value.stock-=Number(dispenseQty.value)
  if(selected.value.stock===0) selected.value.status='已領完'
  emit('busy',false); view.value='complete'
}
function backToList(){ validationError.value=''; selected.value=null; view.value='list' }
function finish(){ backToList() }
function exitModule(){ emit('busy',false); emit('back') }
</script>

<template>
  <section class="own-module">
    <div class="routine-head"><button class="text-back" :disabled="view==='dispensing'" @click="view==='list' ? exitModule() : backToList()">‹ {{ view==='dispensing' ? '取藥進行中' : '返回' }}</button><p>取藥作業</p><h1>病人自備藥取藥</h1></div>

    <div v-if="view==='list'" class="order-list-card">
      <div class="order-list-title"><div><span class="routine-icon">▣</span><div><h2>病人自備藥取藥</h2><p>請選擇要取出的病人自備藥（資料來源：自備藥入庫紀錄）</p></div></div><span>資料更新時間：2025-10-20 11:01</span><button type="button">↻ 重新整理</button></div>
      <div class="own-filters"><label>病房區域<select v-model="query.ward"><option>全部</option><option>一般病房</option><option>內科病房</option><option>外科病房</option></select></label><label>病房<select v-model="query.room"><option>全部</option><option>A103</option><option>A105</option><option>B201</option><option>B202</option><option>C301</option><option>C302</option></select></label><label>床號<input v-model="query.bed" placeholder="請輸入床號"></label><label>病人姓名<input v-model="query.patient" placeholder="請輸入病人姓名"></label><label>藥品名稱<input v-model="query.medicine" placeholder="請輸入藥品名稱"></label><label>狀態<select v-model="query.status"><option>全部</option><option>可取藥</option><option>已領完</option></select></label><button type="button">⌕ 查詢</button></div>
      <p class="result-count">共 {{ filteredRecords.length }} 筆</p>
      <table class="patient-table own-table"><thead><tr><th>病房／床號</th><th>病人姓名</th><th>病歷號</th><th>藥品名稱（劑型／規格）</th><th>入庫日期</th><th>數量（庫存）</th><th>效期</th><th>藥盒</th><th>操作</th></tr></thead><tbody><tr v-for="item in filteredRecords" :key="item.id"><td>{{ item.bed }}</td><td><b>{{ item.patient }}</b></td><td>{{ item.patientId }}</td><td class="medicine-cell"><b>{{ item.generic }}</b><small>{{ item.medicine }}</small></td><td>{{ item.received }}</td><td><b>{{ item.stock }} {{ item.unit }}</b></td><td>{{ item.expiry }}</td><td>{{ item.drawer }} 號</td><td><button :disabled="item.status!=='可取藥'" @click="chooseRecord(item)">{{ item.status==='可取藥' ? '選擇取藥 ›' : '已領完' }}</button></td></tr></tbody></table>
    </div>

    <div v-else-if="view==='confirm'" class="routine-work-card own-confirm">
      <div class="routine-step"><b>1</b><span>確認資料</span><i></i><b>2</b><span>開盒取藥</span><i></i><b>3</b><span>完成記錄</span></div>
      <div class="review-confirm-title"><h2>確認病人自備藥取藥內容</h2><p>請核對病人及藥品資料，並輸入本次實際取藥數量。</p></div>
      <div class="routine-patient"><div><small>病房／床號</small><b>{{ selected.bed }}</b></div><div><small>病人姓名</small><b>{{ selected.patient }}</b></div><div><small>病歷號</small><b>{{ selected.patientId }}</b></div><div><small>入庫紀錄編號</small><b>{{ selected.id }}</b></div></div>
      <div class="own-med-card"><span class="large-own-med">{{ selected.code }}</span><div><h3>{{ selected.medicine }}</h3><p>{{ selected.generic }}</p><dl><div><dt>入庫日期</dt><dd>{{ selected.received }}</dd></div><div><dt>有效期限</dt><dd>{{ selected.expiry }}</dd></div><div><dt>目前庫存</dt><dd>{{ selected.stock }} {{ selected.unit }}</dd></div><div><dt>藥盒位置</dt><dd>{{ selected.drawer }} 號</dd></div></dl></div><label>本次取藥數量<span><input v-model.number="dispenseQty" type="number" min="1" :max="selected.stock" step="1"> {{ selected.unit }}</span><small>最多可取 {{ selected.stock }} {{ selected.unit }}</small></label></div>
      <p v-if="validationError" class="form-error">{{ validationError }}</p>
      <div class="safety-message">按下開始取藥後將開啟 {{ selected.drawer }} 號藥盒；取出藥品並關閉藥盒後，系統才會扣除庫存。</div>
      <div class="routine-actions"><button @click="backToList">返回自備藥清單</button><button class="primary" @click="startDispensing">開始取藥</button></div>
    </div>

    <div v-else-if="view==='dispensing'" class="routine-work-card own-dispensing">
      <div class="routine-step"><b class="done">✓</b><span>確認資料</span><i class="done"></i><b>2</b><span>開盒取藥</span><i></i><b>3</b><span>完成記錄</span></div>
      <div class="routine-patient"><div><small>病房／床號</small><b>{{ selected.bed }}</b></div><div><small>病人姓名</small><b>{{ selected.patient }}</b></div><div><small>病歷號</small><b>{{ selected.patientId }}</b></div><div><small>取藥數量</small><b>{{ dispenseQty }} {{ selected.unit }}</b></div></div>
      <div class="own-open-grid"><div class="large-med-image"><span>{{ selected.code }}</span><small>病人自備藥</small></div><div><p class="overline">目前取藥藥品</p><h2>{{ selected.medicine }}</h2><p>{{ selected.generic }}</p><dl><div><dt>本次取藥</dt><dd>{{ dispenseQty }} {{ selected.unit }}</dd></div><div><dt>取藥前庫存</dt><dd>{{ selected.stock }} {{ selected.unit }}</dd></div><div><dt>藥盒位置</dt><dd>{{ selected.drawer }} 號</dd></div></dl></div><aside><span class="live-dot"></span><small>取藥藥盒狀態</small><h3>{{ selected.drawer }} 號藥盒已開啟</h3><p>請取出 {{ dispenseQty }} {{ selected.unit }}病人自備藥，取出後請關閉藥盒。</p><button @click="closeDrawer">模擬關閉取藥盒</button></aside></div>
    </div>

    <div v-else class="workflow-card complete-card"><div class="complete-icon">✓</div><p class="overline">作業完成</p><h2>病人自備藥取藥已完成</h2><p>{{ selected.patient }}的「{{ selected.medicine }}」已取出 {{ dispenseQty }} {{ selected.unit }}，{{ selected.drawer }} 號藥盒已關閉。</p><div class="own-result"><span>取藥後庫存</span><strong>{{ selected.stock }} {{ selected.unit }}</strong></div><div class="audit-summary"><span>自備藥入庫紀錄已關聯</span><span>取藥人員已記錄</span><span>庫存及藥盒事件已記錄</span></div><button class="primary-action" @click="finish">返回病人自備藥清單</button></div>
  </section>
</template>
