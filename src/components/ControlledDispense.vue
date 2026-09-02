<script setup>
import { computed, ref } from 'vue'

defineProps({ operator: Object })
const emit = defineEmits(['back', 'busy'])

const view = ref('categories')
const supervisorCode = ref('')
const supervisor = ref(null)
const supervisorError = ref('')
const prescriptionCode = ref('')
const prescriptionError = ref('')
const currentIndex = ref(0)
const drawerState = ref('closed')
const returnDrawerState = ref('closed')
const discardedAmount = ref('')
const printStatus = ref('尚未列印')
const completed = ref(false)

const prescription = {
  rxNo: 'RX20251020001', bed: 'A103-2', patientId: '0001234567', patientName: '王小明',
  medicines: [
    { code:'DIAZPAM-5', name:'Diazepam 5mg/tab', chinese:'煩寧錠 5mg', qty:2, unit:'錠', stock:128, drawer:'05', returnDrawer:'12', kind:'tablet', requiresDiscard:false, requiresReturn:true },
    { code:'MORPHINE-10', name:'Morphine Sulfate 10mg/mL inj', chinese:'硫酸嗎啡注射液', qty:1, unit:'支', stock:46, drawer:'08', returnDrawer:'12', kind:'ampoule', requiresDiscard:true, discardUnit:'mL', requiresReturn:true },
    { code:'FENTANYL-25', name:'Fentanyl 25mcg/hr patch', chinese:'吩坦尼穿皮貼片', qty:1, unit:'片', stock:18, drawer:'11', returnDrawer:'12', kind:'patch', requiresDiscard:false, requiresReturn:true }
  ]
}

const medicines = ref([])
const currentMedicine = computed(() => medicines.value[currentIndex.value])
const progressText = computed(() => `${Math.min(currentIndex.value + 1, medicines.value.length)} / ${medicines.value.length}`)

function chooseControlled(){ view.value='supervisor' }
function verifySupervisor(code=supervisorCode.value){
  supervisorError.value=''
  if(code.trim().toUpperCase()!=='S000888'){
    supervisorError.value='驗證失敗：此識別證不具護理主管驗證權限。'
    return
  }
  supervisor.value={ name:'王小美', employeeId:'0888', role:'護理人員主管' }
  supervisorCode.value=''
  view.value='prescription'
}
function fetchPrescription(code=prescriptionCode.value){
  prescriptionError.value=''
  if(code.trim().toUpperCase()!==prescription.rxNo){
    prescriptionError.value='查無處方資料，請確認條碼後重新掃描。'
    return
  }
  medicines.value=prescription.medicines.map(item=>({...item,status:'pending'}))
  prescriptionCode.value=''
  currentIndex.value=0
  view.value='dispensing'
  startMedicine()
}
function startMedicine(){
  const med=currentMedicine.value
  if(!med){ finishAll(); return }
  med.status='processing'
  discardedAmount.value=''
  printStatus.value='列印完成'
  drawerState.value='open'
  emit('busy',true)
}
function closeMedicineDrawer(){
  const med=currentMedicine.value
  if(med.requiresDiscard && (discardedAmount.value==='' || Number(discardedAmount.value)<0)){
    window.alert('請先輸入正確的實際棄藥量。')
    return
  }
  drawerState.value='closed'
  if(med.requiresReturn){ returnDrawerState.value='open'; return }
  completeCurrent()
}
function closeReturnDrawer(){
  returnDrawerState.value='closed'
  completeCurrent()
}
function completeCurrent(){
  currentMedicine.value.status='done'
  if(currentIndex.value < medicines.value.length-1){
    currentIndex.value+=1
    window.setTimeout(startMedicine,500)
  }else finishAll()
}
function finishAll(){
  emit('busy',false)
  completed.value=true
  view.value='complete'
}
function resetToCategories(){
  emit('busy',false)
  view.value='categories'; supervisor.value=null; medicines.value=[]; completed.value=false
}
function statusLabel(status){ return status==='done'?'已完成':status==='processing'?'處理中':'待處理' }
</script>

<template>
  <main class="module-page">
    <div class="module-head">
      <div><button class="text-back" @click="view==='categories' ? emit('back') : resetToCategories()">‹ 返回</button><p>取藥作業</p><h1>{{ view==='categories' ? '請選擇取藥類型' : '管制藥取藥' }}</h1></div>
      <div v-if="supervisor" class="verified-badge">✓ 二次驗證：{{ supervisor.name }}（{{ supervisor.employeeId }}）</div>
    </div>

    <section v-if="view==='categories'" class="dispense-types">
      <button class="type-card featured" @click="chooseControlled"><span class="type-icon">Rx</span><b>管制藥取藥</b><small>需護理主管二次驗證</small><i>開始作業 →</i></button>
      <button class="type-card" disabled><span class="type-icon">＋</span><b>常備藥取藥</b><small>後續模組開放</small></button>
      <button class="type-card" disabled><span class="type-icon">▣</span><b>病人自備藥取藥</b><small>後續模組開放</small></button>
    </section>

    <section v-else-if="view==='supervisor'" class="workflow-card verify-card">
      <div class="stepper"><span class="active">1</span><i></i><span>2</span><i></i><span>3</span></div>
      <div class="large-round-icon">♙</div><p class="overline">步驟 1／3</p><h2>護理主管二次驗證</h2><p>請由具管制藥驗證權限的護理主管刷讀人員識別證。</p>
      <form class="module-form" @submit.prevent="verifySupervisor()"><label>主管識別證 Barcode</label><div><input v-model="supervisorCode" autofocus placeholder="請刷讀主管識別證"><button>驗證</button></div></form>
      <p v-if="supervisorError" class="form-error">{{ supervisorError }}</p>
      <div class="prototype-actions"><span>Prototype：</span><button @click="verifySupervisor('S000888')">模擬主管刷卡</button><button @click="verifySupervisor('N001162')">模擬無驗證權限</button></div>
    </section>

    <section v-else-if="view==='prescription'" class="workflow-card verify-card">
      <div class="stepper"><span class="done">✓</span><i class="done"></i><span class="active">2</span><i></i><span>3</span></div>
      <div class="large-round-icon">▥</div><p class="overline">步驟 2／3</p><h2>請刷讀處方籤條碼</h2><p>系統將透過 HIS API 取得處方資料，並只顯示藥櫃內配置的藥品。</p>
      <form class="module-form" @submit.prevent="fetchPrescription()"><label>處方籤 Barcode</label><div><input v-model="prescriptionCode" autofocus placeholder="請刷讀處方籤條碼"><button>查詢處方</button></div></form>
      <p v-if="prescriptionError" class="form-error">{{ prescriptionError }}</p>
      <div class="prototype-actions"><span>Prototype：</span><button @click="fetchPrescription('RX20251020001')">模擬刷讀處方</button><code>RX20251020001</code></div>
    </section>

    <section v-else-if="view==='dispensing'" class="dispensing-layout">
      <div class="patient-strip"><div><span>病床</span><b>{{ prescription.bed }}</b></div><div><span>病人姓名</span><b>{{ prescription.patientName }}</b></div><div><span>病歷號</span><b>{{ prescription.patientId }}</b></div><div><span>處方單號</span><b>{{ prescription.rxNo }}</b></div><div><span>標籤</span><b class="success-text">✓ {{ printStatus }}</b></div></div>
      <div class="main-med-card">
        <div class="med-photo" :class="currentMedicine.kind"><div class="drug-object"><span>{{ currentMedicine.code }}</span></div><small>藥品示意圖</small></div>
        <div class="med-detail"><div class="med-sequence">目前取藥 · {{ progressText }}</div><h2>{{ currentMedicine.chinese }}</h2><p>{{ currentMedicine.name }}</p><dl><div><dt>應取數量</dt><dd>{{ currentMedicine.qty }} {{ currentMedicine.unit }}</dd></div><div><dt>藥盒位置</dt><dd>{{ currentMedicine.drawer }} 號</dd></div><div><dt>目前庫存</dt><dd>{{ currentMedicine.stock }} {{ currentMedicine.unit }}</dd></div></dl>
          <label v-if="currentMedicine.requiresDiscard" class="discard-field">實際棄藥量 <span><input v-model="discardedAmount" min="0" step="0.1" type="number" placeholder="請輸入"> {{ currentMedicine.discardUnit }}</span></label>
        </div>
        <aside class="drawer-panel"><span class="live-dot"></span><small>取藥藥盒狀態</small><strong>{{ currentMedicine.drawer }} 號藥盒已開啟</strong><p>請取出 {{ currentMedicine.qty }} {{ currentMedicine.unit }}，貼妥標籤後關閉藥盒。</p><button @click="closeMedicineDrawer">模擬關閉取藥盒</button></aside>
      </div>
      <div class="medicine-progress"><h3>本張處方取藥進度</h3><div class="progress-list"><div v-for="(med,index) in medicines" :key="med.code" :class="med.status"><span class="mini-drug">{{ index+1 }}</span><p><b>{{ med.chinese }}</b><small>{{ med.qty }} {{ med.unit }} · {{ med.drawer }}號藥盒</small></p><em>{{ statusLabel(med.status) }}</em></div></div></div>
      <div v-if="returnDrawerState==='open'" class="modal-layer"><section class="return-modal"><div class="return-icon">♻</div><p class="overline">空瓶／包材回收</p><h2>請將本次藥品空瓶放入回收盒</h2><div class="return-med"><b>{{ currentMedicine.chinese }}</b><span>{{ currentMedicine.name }}</span><strong>{{ currentMedicine.qty }} {{ currentMedicine.unit }}</strong></div><div class="drawer-callout"><span class="live-dot"></span><div><small>回收盒已開啟</small><b>{{ currentMedicine.returnDrawer }} 號回收盒</b></div></div><p>放置完成後請關閉回收盒，系統才會開啟下一個藥品的藥盒。</p><button @click="closeReturnDrawer">模擬關閉回收盒</button></section></div>
    </section>

    <section v-else-if="view==='complete'" class="workflow-card complete-card"><div class="complete-icon">✓</div><p class="overline">作業完成</p><h2>管制藥取藥已完成</h2><p>本張處方共完成 {{ medicines.length }} 項藥品，所有取藥盒及回收盒均已關閉。</p><div class="audit-summary"><span>主要操作人員已記錄</span><span>主管二次驗證已記錄</span><span>藥盒事件已記錄</span></div><button class="primary-action" @click="resetToCategories">返回取藥頁面</button></section>
  </main>
</template>
