<script setup>
import { computed, ref } from 'vue'

const emit = defineEmits(['back', 'busy'])
const view = ref('patients')
const query = ref({ ward:'全部', room:'全部', bed:'', name:'', orderType:'全部' })
const selectedPatient = ref(null)
const selectedCodes = ref([])
const currentIndex = ref(0)
const drawerState = ref('closed')
const returnDrawerState = ref('closed')
const discardedAmount = ref('')

const patients = [
  { bed:'A103-2', patientId:'0001234567', name:'王小明', room:'A103', ward:'一般病房', orderType:'長期用藥', count:3, updated:'2025-10-20 09:15' },
  { bed:'A105-1', patientId:'0002345678', name:'李大華', room:'A105', ward:'一般病房', orderType:'長期用藥', count:2, updated:'2025-10-20 08:42' },
  { bed:'B201-1', patientId:'0003456789', name:'張美玲', room:'B201', ward:'內科病房', orderType:'臨時醫囑', count:4, updated:'2025-10-20 07:58' },
  { bed:'B202-2', patientId:'0004567890', name:'陳建宏', room:'B202', ward:'內科病房', orderType:'長期用藥', count:3, updated:'2025-10-19 17:30' },
  { bed:'C301-1', patientId:'0005678901', name:'林志偉', room:'C301', ward:'外科病房', orderType:'長期用藥', count:2, updated:'2025-10-19 16:10' }
]

const orderMedicines = [
  { code:'ACETA-500', chinese:'普拿疼錠 500mg', name:'Acetaminophen 500mg/tab', qty:2, unit:'錠', stock:186, drawer:'03', kind:'tablet', requiresDiscard:false, requiresReturn:false, checked:true },
  { code:'CEFAZ-1000', chinese:'西華樂林注射劑 1g', name:'Cefazolin 1g/vial', qty:1, unit:'瓶', stock:34, drawer:'07', returnDrawer:'15', kind:'ampoule', requiresDiscard:true, discardUnit:'mL', requiresReturn:true, checked:true },
  { code:'SALINE-10', chinese:'生理食鹽水注射液 10mL', name:'Sodium Chloride 0.9% 10mL', qty:1, unit:'支', stock:72, drawer:'09', returnDrawer:'15', kind:'ampoule', requiresDiscard:false, requiresReturn:true, checked:false }
]

const medicines = ref([])
const filteredPatients = computed(() => patients.filter(p =>
  (query.value.ward==='全部'||p.ward===query.value.ward) &&
  (query.value.room==='全部'||p.room===query.value.room) &&
  (!query.value.bed||p.bed.includes(query.value.bed)) &&
  (!query.value.name||p.name.includes(query.value.name)) &&
  (query.value.orderType==='全部'||p.orderType===query.value.orderType)
))
const selectedMedicines = computed(() => medicines.value.filter(m => selectedCodes.value.includes(m.code)))
const currentMedicine = computed(() => selectedMedicines.value[currentIndex.value])

function choosePatient(patient){
  selectedPatient.value=patient
  medicines.value=orderMedicines.map(m=>({...m,status:'pending'}))
  selectedCodes.value=medicines.value.filter(m=>m.checked).map(m=>m.code)
  view.value='select'
}
function goReview(){ if(selectedCodes.value.length) view.value='review' }
function startDispensing(){ currentIndex.value=0; view.value='dispensing'; startMedicine() }
function startMedicine(){
  const med=currentMedicine.value
  if(!med){ finishAll(); return }
  med.status='processing'; discardedAmount.value=''; drawerState.value='open'; emit('busy',true)
}
function closeMedicineDrawer(){
  const med=currentMedicine.value
  if(med.requiresDiscard && (discardedAmount.value==='' || Number(discardedAmount.value)<0)){ window.alert('請先輸入正確的實際棄藥量。'); return }
  drawerState.value='closed'
  if(med.requiresReturn){ returnDrawerState.value='open'; return }
  completeCurrent()
}
function closeReturnDrawer(){ returnDrawerState.value='closed'; completeCurrent() }
function completeCurrent(){
  currentMedicine.value.status='done'
  if(currentIndex.value<selectedMedicines.value.length-1){ currentIndex.value+=1; window.setTimeout(startMedicine,500) } else finishAll()
}
function finishAll(){ emit('busy',false); view.value='complete' }
function reset(){ emit('busy',false); emit('back') }
function statusLabel(status){ return status==='done'?'已完成':status==='processing'?'處理中':'待處理' }
</script>

<template>
  <section class="routine-module">
    <div class="routine-head"><button class="text-back" :disabled="view==='dispensing'" @click="view==='patients' ? reset() : view='patients'">‹ {{ view==='dispensing' ? '取藥進行中' : '返回' }}</button><p>取藥作業</p><h1>常備藥取藥</h1></div>

    <div v-if="view==='patients'" class="order-list-card">
      <div class="order-list-title"><div><span class="routine-icon">▣</span><div><h2>常備藥取藥</h2><p>請選擇要取藥的病人（資料來源：HIS 醫囑彙整）</p></div></div><span>資料更新時間：2025-10-20 11:01</span><button>↻ 重新整理</button></div>
      <div class="order-filters"><label>病房區域<select v-model="query.ward"><option>全部</option><option>一般病房</option><option>內科病房</option><option>外科病房</option></select></label><label>病房<select v-model="query.room"><option>全部</option><option>A103</option><option>A105</option><option>B201</option><option>B202</option><option>C301</option></select></label><label>床號<input v-model="query.bed" placeholder="請輸入床號"></label><label>病人姓名<input v-model="query.name" placeholder="請輸入病人姓名"></label><label>醫囑類型<select v-model="query.orderType"><option>全部</option><option>長期用藥</option><option>臨時醫囑</option></select></label><button>⌕ 查詢</button></div>
      <p class="result-count">共 {{ filteredPatients.length }} 筆</p>
      <table class="patient-table"><thead><tr><th>床號</th><th>病歷號</th><th>病人姓名</th><th>醫囑類型</th><th>可取藥品項數</th><th>最後醫囑時間</th><th>操作</th></tr></thead><tbody><tr v-for="patient in filteredPatients" :key="patient.patientId"><td>{{ patient.bed }}</td><td>{{ patient.patientId }}</td><td><b>{{ patient.name }}</b></td><td>{{ patient.orderType }}</td><td>{{ patient.count }} 項</td><td>{{ patient.updated }}</td><td><button @click="choosePatient(patient)">選擇取藥 ›</button></td></tr></tbody></table>
    </div>

    <div v-else-if="view==='select'" class="routine-work-card">
      <div class="routine-step"><b>1</b><span>選擇藥品</span><i></i><b>2</b><span>確認內容</span><i></i><b>3</b><span>依序取藥</span></div>
      <div class="routine-patient"><div><small>床號</small><b>{{ selectedPatient.bed }}</b></div><div><small>病人姓名</small><b>{{ selectedPatient.name }}</b></div><div><small>病歷號</small><b>{{ selectedPatient.patientId }}</b></div><div><small>醫囑類型</small><b>{{ selectedPatient.orderType }}</b></div></div>
      <div class="selection-title"><div><h2>選擇本次要取的藥品</h2><p>取藥數量依醫囑帶入，不開放修改。</p></div><strong>已選擇 {{ selectedCodes.length }} 項</strong></div>
      <div class="select-med-list"><label v-for="med in medicines" :key="med.code" :class="{selected:selectedCodes.includes(med.code)}"><input v-model="selectedCodes" type="checkbox" :value="med.code"><span class="routine-med-image">{{ med.code }}</span><span><b>{{ med.chinese }}</b><small>{{ med.name }}</small><em>{{ med.code }}</em></span><dl><div><dt>醫囑取藥量</dt><dd>{{ med.qty }} {{ med.unit }}</dd></div><div><dt>藥盒</dt><dd>{{ med.drawer }} 號</dd></div><div><dt>庫存</dt><dd>{{ med.stock }} {{ med.unit }}</dd></div><div><dt>作業需求</dt><dd>{{ med.requiresDiscard ? '需填棄藥量' : '一般取藥' }}{{ med.requiresReturn ? '／需回收' : '' }}</dd></div></dl></label></div>
      <div class="routine-actions"><button @click="view='patients'">返回病人清單</button><button class="primary" :disabled="!selectedCodes.length" @click="goReview">確認取藥內容（{{ selectedCodes.length }}）</button></div>
    </div>

    <div v-else-if="view==='review'" class="routine-work-card">
      <div class="routine-step"><b class="done">✓</b><span>選擇藥品</span><i class="done"></i><b>2</b><span>確認內容</span><i></i><b>3</b><span>依序取藥</span></div>
      <div class="review-confirm-title"><h2>確認本次常備藥取藥內容</h2><p>開始後將依序列印標籤並開啟藥盒，請再次確認病人及藥品資料。</p></div>
      <div class="routine-patient"><div><small>床號</small><b>{{ selectedPatient.bed }}</b></div><div><small>病人姓名</small><b>{{ selectedPatient.name }}</b></div><div><small>病歷號</small><b>{{ selectedPatient.patientId }}</b></div><div><small>取藥品項</small><b>{{ selectedMedicines.length }} 項</b></div></div>
      <div class="confirm-list"><div v-for="(med,index) in selectedMedicines" :key="med.code"><span>{{ index+1 }}</span><p><b>{{ med.chinese }}</b><small>{{ med.name }}</small></p><strong>{{ med.qty }} {{ med.unit }}</strong><em>{{ med.drawer }} 號藥盒</em></div></div>
      <div class="safety-message">開始取藥後，一次只會開啟一個藥盒；完成目前藥品及必要的空瓶回收後，才會進入下一項。</div>
      <div class="routine-actions"><button @click="view='select'">返回修改</button><button class="primary" @click="startDispensing">開始取藥</button></div>
    </div>

    <div v-else-if="view==='dispensing'" class="routine-work-card dispensing-routine">
      <div class="routine-step"><b class="done">✓</b><span>選擇藥品</span><i class="done"></i><b class="done">✓</b><span>確認內容</span><i class="done"></i><b>3</b><span>依序取藥</span></div>
      <div class="routine-patient"><div><small>床號</small><b>{{ selectedPatient.bed }}</b></div><div><small>病人姓名</small><b>{{ selectedPatient.name }}</b></div><div><small>病歷號</small><b>{{ selectedPatient.patientId }}</b></div><div><small>標籤狀態</small><b class="success-text">✓ 本項列印完成</b></div></div>
      <div class="routine-dispense-grid"><div class="large-med-image" :class="currentMedicine.kind"><span>{{ currentMedicine.code }}</span><small>藥品示意圖</small></div><div class="routine-med-detail"><em>目前取藥 {{ currentIndex+1 }}／{{ selectedMedicines.length }}</em><h2>{{ currentMedicine.chinese }}</h2><p>{{ currentMedicine.name }}</p><dl><div><dt>醫囑取藥量</dt><dd>{{ currentMedicine.qty }} {{ currentMedicine.unit }}</dd></div><div><dt>藥盒位置</dt><dd>{{ currentMedicine.drawer }} 號</dd></div><div><dt>目前庫存</dt><dd>{{ currentMedicine.stock }} {{ currentMedicine.unit }}</dd></div></dl><label v-if="currentMedicine.requiresDiscard">實際棄藥量 <span><input v-model="discardedAmount" type="number" min="0" step="0.1" placeholder="請輸入"> {{ currentMedicine.discardUnit }}</span></label></div><aside><span class="live-dot"></span><small>取藥藥盒狀態</small><h3>{{ currentMedicine.drawer }} 號藥盒已開啟</h3><p>請取出 {{ currentMedicine.qty }} {{ currentMedicine.unit }}，貼妥標籤後關閉藥盒。</p><button @click="closeMedicineDrawer">模擬關閉取藥盒</button></aside></div>
      <div class="routine-progress"><div v-for="(med,index) in selectedMedicines" :key="med.code" :class="med.status"><span>{{ index+1 }}</span><p><b>{{ med.chinese }}</b><small>{{ med.qty }} {{ med.unit }}</small></p><em>{{ statusLabel(med.status) }}</em></div></div>
      <div v-if="returnDrawerState==='open'" class="modal-layer"><section class="return-modal"><div class="return-icon">♻</div><p class="overline">空瓶／容器回收</p><h2>請將本次藥品空瓶放入回收盒</h2><div class="return-med"><b>{{ currentMedicine.chinese }}</b><span>{{ currentMedicine.name }}</span><strong>{{ currentMedicine.qty }} {{ currentMedicine.unit }}</strong></div><div class="drawer-callout"><span class="live-dot"></span><div><small>回收盒已開啟</small><b>{{ currentMedicine.returnDrawer }} 號回收盒</b></div></div><p>關閉回收盒後，系統才會進入下一項藥品。</p><button @click="closeReturnDrawer">模擬關閉回收盒</button></section></div>
    </div>

    <div v-else class="workflow-card complete-card"><div class="complete-icon">✓</div><p class="overline">作業完成</p><h2>常備藥取藥已完成</h2><p>{{ selectedPatient.name }}本次共完成 {{ selectedMedicines.length }} 項藥品，所有藥盒均已關閉。</p><div class="audit-summary"><span>醫囑資料已記錄</span><span>取藥人員已記錄</span><span>藥盒事件已記錄</span></div><button class="primary-action" @click="reset">返回取藥頁面</button></div>
  </section>
</template>
