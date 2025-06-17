<script lang="ts" setup>
import { toggleDark } from "~/composables";
import { ref, defineEmits } from 'vue';

const examType = ref('psm');
const emit = defineEmits(['exam-change']);
function handleExamChange(val: string) {
  examType.value = val;
  emit('exam-change', val);
}
</script>

<template>
  <el-menu class="el-menu-demo" mode="horizontal">
    <el-menu-item index="1">Element Plus</el-menu-item>
    <el-menu-item index="2">
      <el-select v-model="examType" style="width: 140px" @change="handleExamChange">
        <el-option label="PSM" value="psm" />
        <el-option label="Security+" value="security+" />
      </el-select>
    </el-menu-item>
    <el-menu-item h="full" @click="toggleDark()">
      <button
        class="border-none w-full bg-transparent cursor-pointer"
        style="height: var(--ep-menu-item-height)"
      >
        <i inline-flex i="dark:ep-moon ep-sunny" />
      </button>
    </el-menu-item>
  </el-menu>
</template>
