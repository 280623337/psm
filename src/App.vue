<template>
  <el-config-provider namespace="ep">
    <BaseHeader @exam-change="onExamChange" />
    <div class="flex main-container">
      <!-- <BaseSide /> -->
      <div w="full" py="4">
        <!-- <Logos my="4" /> -->
        <HelloWorld :examType="examType" msg="Hello Vue 3 + Element Plus + Vite" />
      </div>
    </div>
  </el-config-provider>
</template>

<script setup lang="ts">
import { ref } from 'vue';
const examType = ref('psm');
function onExamChange(val: string) {
  examType.value = val;
}
</script>

<style>
#app {
  text-align: center;
  color: var(--ep-text-color-primary);
}

.main-container {
  height: calc(100vh - var(--ep-menu-item-height) - 3px);
}
</style>
