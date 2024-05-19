<template>
  <el-row :gutter="24">
    <el-col :span="20" :offset="2">
      <el-button v-if="testMode === false" @click="setMode" type="info">Switch To Test Mode</el-button>
      <el-button v-else type="danger"  @click="setMode" >Switch To Read Mode</el-button>
      <el-button v-if="testMode" @click="verifyAnswer">Submit</el-button>
      <el-text v-if="testMode && score >= 0" size="large"><h2>Score: {{ score }}</h2></el-text>
      <br/>
      <div v-if="testMode === false" style="text-align: left;word-wrap: break-word;" v-for="(question, index) in questionList" :key="index">
        <div style="padding:10px;margin-bottom: 10px;">
          <h2>{{ index+1 + "." + question.question }}</h2>
          <div v-for="(option, oIndex) in question.options" :key="oIndex">
            <el-checkbox class="checkboxitem"   v-model="option.correct" :label="calculateCharFromNumber(oIndex)+ '. ' +  option.content" size="large"/> 
          </div>
        </div>
        <br/>
      </div>
      <div v-else style="text-align: left;word-wrap: break-word;" v-for="(question, index) in testQuestionList" >
        <div v-if="result[index]" style="padding:10px; border: 1px solid red;margin-bottom: 5px;">
          <h2>{{ index+1 + "." + question.question }}</h2>
          <div v-for="(option, oIndex) in question.options" :key="oIndex">
            <el-checkbox class="checkboxitem"   v-model="option.correct" :label="calculateCharFromNumber(oIndex)+ '. ' + option.content "  size="large"/>
          </div>
          Answer：{{ result[index]}}
        </div>
        <div v-else style="padding:10px;margin-bottom: 5px;">
          <h2>{{ index+1 + "." + question.question }}</h2>
          <div v-for="(option, oIndex) in question.options" :key="oIndex">
            <el-checkbox class="checkboxitem"   v-model="option.correct" :label="calculateCharFromNumber(oIndex)+ '. ' + option.content "  size="large"/>
          </div>
        </div>
        <br/>
      </div>
      <el-button v-if="testMode" @click="verifyAnswer">Submit</el-button>
    </el-col>
  </el-row>
</template>

<script setup lang="ts">
import { ref } from "vue";
import axios from 'axios';
import test from "node:test";

interface Option {
  content: string,
  correct: boolean,
  
}

interface Question {
  question: string;
  options: Option[];
}

const questionList = ref<Question[]>([])
const answer = ref<Question[]>([])
const testMode = ref(false)
const score = ref(-1)
const result = ref<string[]>([])
const testQuestionList = ref<Question[]>([])
const answerQuestionList = ref<Question[]>([])
load_data()

function load_data() {
  // 使用fetch函数读取本地JSON文件
  axios.get('/data/all.json')
  .then(response => {
      console.log(response)
      questionList.value = response.data;
      answer.value = response.data;
  })
  .catch(error => {
    console.error('Error reading JSON file:', error);
  });
}

function generateRandomNumbers(count: number = 10, min: number = 1, max: number = 100) : number[] {
  if (count > (max - min + 1)) {
    throw new Error("The range is not large enough to generate " + count + " unique numbers.");
  }

  const numbers = new Set<number>();

  while (numbers.size < count) {
    const randomNumber = Math.floor(Math.random() * (max - min + 1)) + min;
    numbers.add(randomNumber);
  }

  return Array.from(numbers);
}

function deepCopy(obj: Question) {
  return JSON.parse(JSON.stringify(obj));
}

function setMode() {
  testMode.value = !testMode.value
  score.value = -1
  if (testMode.value) {
    testQuestionList.value = []
    answerQuestionList.value = []
    result.value = []
    const numbers: number[] = generateRandomNumbers(3, 0, questionList.value.length)

    for(let i=0; i<numbers.length;i++) {
      const tmp:Question = deepCopy(questionList.value[numbers[i]])
      for(let j=0; j<tmp.options.length;j++) {
        console.log(tmp.options)
        console.log(tmp.options[j].content)
        tmp.options[j].correct = false 
      }
      testQuestionList.value.push(tmp)
      answerQuestionList.value.push(deepCopy(questionList.value[numbers[i]]))
      result.value.push("")
    }
  }
}
function calculateCharFromNumber(num: number) :string {
  if (num < 0 || num > 25) {
    throw new Error("The number should be between 0 and 25.");
  }

  const charCode = num + 65;
  return String.fromCharCode(charCode);
}

function verifyAnswer() {
  // 使用fetch函数读取本地JSON文件
  console.log("start to verify answer")
  const tmp = ref<string[]>([])
  const count = ref(0)
  for(let i=0; i<testQuestionList.value.length;i++){
    const r = ref<string[]>([])
    let options = testQuestionList.value[i].options
    let flag = true
    for(let j=0;j<options.length;j++) {
      if (options[j].correct != answerQuestionList.value[i].options[j].correct) {
        flag = false
        break
      } 
    }
    if (!flag) {
      let options = answerQuestionList.value[i].options
      for(let j:number=0;j<options.length;j++) {
        if (options[j].correct) {
          r.value.push(calculateCharFromNumber(j))
        }
      }
    } else {
      count.value++
    }
    tmp.value.push(r.value.join(","))
  }
  const tmpScore:number = count.value / testQuestionList.value.length * 100
  score.value = parseFloat(tmpScore.toFixed(2))
  result.value = tmp.value
}
</script>

<style>
.ep-button {
  margin: 4px;
}
.ep-button + .ep-button {
  margin-left: 0;
  margin: 4px;
}
.checkboxitem {
  /* word-wrap: break-all;
  width: 100%;
  white-space: pre-line; */
  white-space: normal;
  word-break: break-word;
  line-height: 15;
  margin-bottom: 5px;
  font-size: 30px;
    /* white-space: pre-line;
  
  overflow: hidden;
  line-height: 30px;
  height: 120px;  */
}
.ep-checkbox.ep-checkbox--large .ep-checkbox__label {
    font-size: 16px;
    line-height: 1.2;
}
</style>
