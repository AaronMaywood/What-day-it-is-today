<script setup>
import { ref } from 'vue'
import WhatDay from './components/WhatDay.vue'

const now = new Date()
const monthDay = (now.getMonth()+1)+"月"+now.getDate()+"日 "
const today = ref(monthDay)
</script>

<template>
  <main>
    <h1>今日はなんの日？（{{today}}）</h1>
	  <WhatDay></WhatDay>
  </main>
</template>
