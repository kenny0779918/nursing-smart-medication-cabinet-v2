<script setup>
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'
import SystemHeader from './components/SystemHeader.vue'
const page = ref('login'), user = ref(null), barcode = ref(''), errorMessage = ref(''), returnCountdown = ref(3)
let returnTimer, inactivityTimer
const functions = [
  { key:'DISPENSE',label:'取藥',icon:'◒' },{ key:'STOCK_IN',label:'入庫',icon:'▣' },{ key:'RETURN',label:'退藥',icon:'◓' },{ key:'INVENTORY',label:'盤點',icon:'☷' },
  { key:'AUDIT',label:'盤點稽核',icon:'☑' },{ key:'REPLENISH',label:'補缺藥',icon:'✚' },{ key:'CLOSE_WARD',label:'關病房',icon:'▥' },{ key:'SETTINGS',label:'系統設定',icon:'⚙' }
]
const roleProfiles = {
  NURSE:{ barcode:'N001162',name:'簡祐杰',employeeId:'1162',role:'護理人員',permissions:['DISPENSE','STOCK_IN','RETURN','INVENTORY','REPLENISH'] },
  SUPERVISOR:{ barcode:'S000888',name:'王小美',employeeId:'0888',role:'護理人員主管',permissions:functions.map(item=>item.key) }
}
const roleLabel = computed(() => user.value?.role ?? '')
function loginByBarcode(inputBarcode=barcode.value){ const code=inputBarcode.trim().toUpperCase(); const matched=Object.values(roleProfiles).find(item=>item.barcode===code); if(!matched){ showAccessDenied('查無人員資料，或此人員無智慧藥櫃操作權限。'); return } user.value={...matched}; barcode.value=''; page.value='home'; resetInactivityTimer() }
function showAccessDenied(message){ errorMessage.value=message; returnCountdown.value=3; page.value='denied'; window.clearInterval(returnTimer); returnTimer=window.setInterval(()=>{ returnCountdown.value-=1; if(returnCountdown.value<=0){ window.clearInterval(returnTimer); returnToLogin() } },1000) }
function returnToLogin(){ page.value='login'; barcode.value=''; errorMessage.value=''; returnCountdown.value=3 }
function logout(){ user.value=null; window.clearTimeout(inactivityTimer); returnToLogin() }
function resetInactivityTimer(){ if(!user.value)return; window.clearTimeout(inactivityTimer); inactivityTimer=window.setTimeout(logout,5*60*1000) }
function isAllowed(key){ return user.value?.permissions.includes(key) }
function openFunction(item){ resetInactivityTimer(); if(!isAllowed(item.key))return; window.alert(`${item.label}功能將於後續模組開放。`) }
function activityHandler(){ resetInactivityTimer() }
onMounted(()=>['click','keydown','mousemove','touchstart'].forEach(event=>window.addEventListener(event,activityHandler)))
onBeforeUnmount(()=>{ ['click','keydown','mousemove','touchstart'].forEach(event=>window.removeEventListener(event,activityHandler)); window.clearInterval(returnTimer); window.clearTimeout(inactivityTimer) })
</script>
<template>
  <div class="app-shell">
    <SystemHeader :user="user" @home="page='home'" @logout="logout" />
    <main v-if="page==='login'" class="auth-page"><section class="scan-card">
      <div class="scan-title-icon">▤</div><h1>請刷讀人員識別證</h1><p class="scan-subtitle">請將員工識別證條碼對準條碼機掃描，進行身分驗證</p>
      <div class="scanner-illustration"><div class="scanner-gun">▰</div><div class="laser"></div><div class="id-card"><span>●</span><b>▤</b><em>▥▥▥▥▥</em></div></div>
      <form class="barcode-form" @submit.prevent="loginByBarcode()"><label for="barcode">Barcode輸入</label><div><input id="barcode" v-model="barcode" autocomplete="off" autofocus placeholder="掃描後自動登入，或輸入測試條碼"><button>驗證</button></div></form>
      <div class="prototype-tools"><span>Prototype測試：</span><button @click="loginByBarcode('N001162')">護理人員</button><button @click="loginByBarcode('S000888')">護理主管</button><button class="denied" @click="loginByBarcode('INVALID')">無權限人員</button></div>
    </section></main>
    <main v-else-if="page==='denied'" class="auth-page"><section class="denied-card"><div class="denied-icon">!</div><p class="eyebrow">身分驗證失敗</p><h1>人員無操作權限</h1><p>{{ errorMessage }}</p><div class="countdown">{{ returnCountdown }}秒後自動返回識別證刷讀畫面</div></section></main>
    <main v-else class="home-page"><section class="welcome-row"><div><p>目前角色：{{ roleLabel }}</p><h1>您好，{{ user.name }}</h1></div><span>閒置5分鐘將自動登出</span></section>
      <section class="function-grid"><button v-for="item in functions" :key="item.key" class="function-card" :class="{disabled:!isAllowed(item.key)}" :disabled="!isAllowed(item.key)" @click="openFunction(item)"><div class="function-icon">{{ item.icon }}</div><strong>{{ item.label }}</strong><span v-if="!isAllowed(item.key)">無操作權限</span><i></i></button></section>
    </main>
  </div>
</template>
