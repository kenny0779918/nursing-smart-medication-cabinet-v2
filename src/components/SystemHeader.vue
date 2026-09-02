<script setup>
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'
defineProps({ user: Object })
defineEmits(['home', 'logout'])
const now = ref(new Date())
let clockTimer
const dateTime = computed(() => new Intl.DateTimeFormat('zh-TW', { year:'numeric', month:'2-digit', day:'2-digit', hour:'2-digit', minute:'2-digit', second:'2-digit', hour12:false }).format(now.value))
onMounted(() => { clockTimer = window.setInterval(() => { now.value = new Date() }, 1000) })
onBeforeUnmount(() => window.clearInterval(clockTimer))
const statuses = ['資料庫連線','ADC控制器','Barcode Scanner']
</script>
<template>
  <header class="system-header">
    <div class="brand-block"><div class="brand-mark">億</div><div><strong>億威電子</strong><span>EMMIT</span></div></div>
    <div class="header-time"><span>◷</span>{{ dateTime }}</div>
    <div class="operator-block"><span class="operator-icon">●</span><span>操作人員：</span><strong v-if="user">{{ user.name }}（{{ user.employeeId }}）</strong><strong v-else class="not-login">尚未登入</strong></div>
    <div class="header-actions"><button v-if="user" class="header-button" @click="$emit('home')">⌂&nbsp; 回首頁</button><button v-if="user" class="header-button" @click="$emit('logout')">⇥&nbsp; 登出</button></div>
    <div class="system-status"><span v-for="item in statuses" :key="item"><i class="status-dot"></i>{{ item }}</span></div>
  </header>
</template>
