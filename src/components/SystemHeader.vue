<script setup>
defineProps({ user: Object })
defineEmits(['home', 'logout'])
</script>

<template>
  <header class="system-header">
    <div class="brand-block">
      <div class="brand-mark">✚</div>
      <div>
        <strong>護理部智慧藥櫃</strong>
        <span>Nursing Smart Medication Cabinet</span>
      </div>
    </div>
    <div class="header-right">
      <div class="system-status">
        <span><i class="status-dot green"></i>溫溼度 23.6°C / 54%</span>
        <span><i class="status-dot green"></i>藥櫃正常</span>
        <span><i class="status-dot green"></i>資料庫連線</span>
      </div>
      <div v-if="user" class="user-block">
        <span class="avatar">{{ user.name.slice(0, 1) }}</span>
        <div><strong>{{ user.name }}</strong><small>{{ user.unit }}</small></div>
      </div>
      <button v-if="user" class="header-button" @click="$emit('home')">回首頁</button>
      <button v-if="user" class="header-button danger" @click="$emit('logout')">登出</button>
    </div>
  </header>
</template>
