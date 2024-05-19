



<script setup lang="ts">
import axios from 'axios';
import { ref } from 'vue'

interface Option {
  content: string,
  correct: boolean,
}

interface Question {
  question: string;
  options: Option[];
}

const questionList = ref<Question[]>([])
await load_data()
const test = "xxx"

async  function load_data() {
  // 使用fetch函数读取本地JSON文件
  axios.get('/data/all.json')
  .then(response => {
      console.log(response)
      questionList.value = response.data;
      // console.log(questionList)
      // questionList = JSON.parse(jsonData);
      // console.log(questionList[0].question)
  })
  .catch(error => {
    console.error('Error reading JSON file:', error);
  });
  console.log("xxxxxxxxxxxx")
  
}
</script>

<template>
  xxxx
  {{  test }}
  <div v-for="(question, index) in questionList" :key="index">

    {{ question.question }}
  </div>
</template>

<style >
.ep-button {
  margin: 4px;
}
.ep-button + .ep-button {
  margin-left: 0;
  margin: 4px;
}
</style>
