<template>
  <el-row :gutter="24" justify="center">
    <el-col :xs="24" :sm="20" :md="16" :lg="12" :xl="10" class="center-col">
      <el-button v-if="testMode === false" @click="setMode" type="info">Switch To Test Mode</el-button>
      <el-button v-else type="danger"  @click="setMode" >Switch To Read Mode</el-button>
      <el-input-number v-if="testMode" v-model="num" :min="1" :max="100" @change="loadTestData" />
      <el-button v-if="testMode && score >= 0"  @click="loadTestData">Retry</el-button>
      <el-text v-if="testMode && score >= 0" ><h2>Score: {{ score }}</h2></el-text>
      <br/>
      <div v-if="testMode === false" style="text-align: left;word-wrap: break-word;" v-for="(question, index) in questionList" :key="index">
        <div style="padding:10px;margin-bottom: 10px;">
          <h2>{{ index+1 + "." + question.question }}</h2>
          <div v-for="(option, oIndex) in question.options" :key="oIndex">
            <el-checkbox class="checkboxitem" v-if="option.correct"  v-model="option.correct" :label="calculateCharFromNumber(oIndex)+ '. ' +  option.content" size="large" /> 
            <el-checkbox class="checkboxitem" v-else :disabled="true" v-model="option.correct" :label="calculateCharFromNumber(oIndex)+ '. ' +  option.content" size="large" /> 
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
      <el-button v-if="testMode && score < 0" @click="verifyAnswer" type="info">Submit</el-button>
    </el-col>
  </el-row>
</template>

<script setup lang="ts">
import { ref, watch, defineProps } from "vue";
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

const props = defineProps<{ examType: string }>();

const questionList = ref<Question[]>([])
const answer = ref<Question[]>([])
const testMode = ref(false)
const score = ref(-1)
const result = ref<string[]>([])
const testQuestionList = ref<Question[]>([])
const answerQuestionList = ref<Question[]>([])
const num = ref(10)

function load_data() {
  let file = '/data/all.json';
  if (props.examType === 'security+') file = '/data/security+.json';
  axios.get(file)
  .then(response => {
      questionList.value = response.data;
      answer.value = response.data;
  })
  .catch(error => {
    console.error('Error reading JSON file:', error);
  });
}

watch(() => props.examType, () => {
  load_data();
  // 切换题库时重置模式和分数
  testMode.value = false;
  score.value = -1;
});

load_data()

function handleChange() {

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
  loadTestData()
}

function loadTestData() {
  score.value = -1
  if (testMode.value) {
    testQuestionList.value = []
    answerQuestionList.value = []
    result.value = []
    const numbers: number[] = generateRandomNumbers(num.value, 0, questionList.value.length)

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
body, #app, html {
  background: #fdf6e3 !important;
}
:root.dark body, :root.dark #app, :root.dark html {
  background: #181818 !important;
}
.center-col {
  margin: 0 auto;
  background: #fdf6e3; /* 书本阅读风格淡黄色 */
  border-radius: 12px;
  box-shadow: 0 2px 16px 0 rgba(0,0,0,0.04);
  padding: 32px 24px 24px 24px;
  min-height: 80vh;
}
:root.dark .center-col {
  background: #232323 !important;
}

.ep-button {
  margin: 4px;
}
.ep-button + .ep-button {
  margin-left: 0;
  margin: 4px;
}
.checkboxitem {
  white-space: normal;
  word-break: break-word;
  margin-bottom: 30px;
  display: flex ;
}
.ep-checkbox.ep-checkbox--large .ep-checkbox__label {
    font-size: 16px;
}
</style>
