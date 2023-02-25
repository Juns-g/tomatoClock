<template>
  <div class="container">
    <div class="outerBox">
      <div class="clockBox">
        <h1>{{ clock.title }}</h1>
        <!-- ! 一次学1h及以上是不正确的 -->
        <!-- <h1>{{ clock.hour }}:{{ clock.minute }}:{{ clock.second }}</h1> -->
        <h1>{{ clock.minute }}:{{ clock.second }}</h1>
        <!-- TODO 总时间，之后删了 -->
        <h1>{{ clock.workTotalTime }}</h1>
      </div>
      <div class="buttonBox">
        <el-button
          class="startButton"
          type="success"
          size="large"
          v-show="!clock.isStart && !clock.breakStatus"
          round
          @click="workStart()"
        >
          开 始
        </el-button>
        <el-button
          class="startBreakButton"
          type="warning"
          size="large"
          v-show="!clock.isStart && clock.breakStatus"
          round
          @click="breakTimer()"
        >
          开 始
        </el-button>
        <el-button
          class="pauseButton"
          type="success"
          size="large"
          v-show="!clock.isPause && clock.isStart"
          round
          @click="pauseTimer()"
        >
          暂 停
        </el-button>
        <el-button
          class="continueButton"
          type="success"
          size="large"
          v-show="clock.isPause && clock.isStart"
          round
          @click="continueTimer()"
        >
          继 续
        </el-button>
        <el-button
          class="resetButton"
          type="success"
          size="large"
          round
          @click="resetTimer()"
        >
          重 置
        </el-button>
        <!-- <el-button
          type="warning"
          size="large"
          :default="111"
          round
          @click="test()"
        >
          测试
        </el-button> -->
        <!--
        <n-button type="warning" secondary round @click="isTimerExist()"
          >定时器存在吗？</n-button
        > -->
      </div>
      <div class="inputBox">
        <!-- ! 一次学1h及以上是不正确的 -->
        <!-- <el-input-number
          v-model="clock.hourInput"
          size="large"
          :min="0"
          :max="24"
        /> -->
        <div class="workButton">
          <span>专注时间: </span>
          <el-input-number
            v-model="clock.workMinuteInput"
            :disable="clock.isNumberInput"
            size="large"
            :min="0"
            :max="60"
          />
          <span> min </span>
        </div>
        <div class="breakButton">
          <!-- 最多休息半小时 -->
          <span>休息时间: </span>
          <el-input-number
            v-model="clock.breakMinuteInput"
            :disable="clock.isNumberInput"
            size="large"
            :min="0"
            :max="30"
          />
          <span> min</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, reactive, watch } from "vue";
import { NButton } from "naive-ui";

let clock = reactive({
  // 展示的时间
  hour: 0,
  minute: 0,
  second: 0,
  // 输入的时间
  // hourInput: 0,
  // 一次专注只准你多少分钟
  workMinuteInput: 0.2,
  breakMinuteInput: 0.1,
  // secondInput: 0,
  workTotalTime: 0,
  breakTotalTime: 0,
  title: "开始专注吧",
  text: [
    "开始专注吧",
    "继续专注吧",
    "正在专注中",
    "休息时间到",
    "正在休息中",
    "休息结束了，继续专注吧",
  ],
  timer: null,
  breakStatus: false,
  isPause: false,
  isStart: false,
  isNumberInput: true,
});

// 根据输入的时分秒计算总时间
/* watch(
  [() => clock.hourInput, () => clock.workMinuteInput, () => clock.secondInput],
  (newValue, oldValue) => {
    // console.log("oldValue :>> ", oldValue);
    // console.log("newValue :>> ", newValue);
    clock.workTotalTime =
      clock.hourInput * 3600 + clock.workMinuteInput * 60 + clock.secondInput;
  }
); */

// 根据输入的分钟计算总时间
/* watch([() => clock.workMinuteInput, () => clock.breakMinuteInput], () => {
  // console.log("oldValue :>> ", oldValue);
  // console.log("newValue :>> ", newValue);
  clock.workTotalTime = clock.workMinuteInput * 60;
  clock.breakTotalTime = clock.breakMinuteInput * 60;
}); */

// 根据总时间计算出显示的时间
watch(
  () => clock.workTotalTime,
  (newValue, oldValue) => {
    // console.log("oldValue :>> ", oldValue);
    // console.log("newValue :>> ", newValue);
    // clock.hour = Math.floor(clock.workTotalTime / 3600);'
    clock.minute = Math.floor(clock.workTotalTime / 60) % 60;
    clock.second = pad(clock.workTotalTime % 60);
  }
);

watch(
  () => clock.breakTotalTime,
  (newValue, oldValue) => {
    clock.minute = Math.floor(clock.breakTotalTime / 60) % 60;
    clock.second = pad(clock.breakTotalTime % 60);
  }
);

// 使用如下的计算属性会导致一开始不显示数字，还不知道why
/* clock.hour = computed(() => {
  Math.floor(clock.workTotalTime / 3600);
});
clock.minute = computed(() => {
  Math.floor(clock.workTotalTime / 60) % 60;
});
clock.second = computed(() => {
  clock.workTotalTime % 60;
}); */

// 秒 转 时分秒
function workTotalTime2time(workTotalTime) {
  let hour = Math.floor(workTotalTime / 3600);
  let minute = Math.floor(workTotalTime / 60) % 60;
  let second = workTotalTime % 60;
  clock.hour = hour;
  clock.minute = minute;
  clock.second = second;
}

function workStart() {
  clock.workTotalTime = clock.workMinuteInput * 60;
  console.log("start清除了定时器");
  clearInterval(clock.timer);
  clock.timer = setInterval(countdown, 1000);
  if (clock.workTotalTime) {
    clock.title = clock.text[2];
  }
  console.log("start运行了，总时间是 :>> ", clock.workTotalTime);
  clock.isStart = true;
  clock.isNumberInput = false;
}

function clear() {
  clearInterval(clock.timer);
  console.log("clear清除了定时器 :>> ", clock.timer);
  clock.workTotalTime = 0;
  console.log("总时间清零了 :>> ", clock.workTotalTime);
  clock.title = clock.text[0];
}

function pauseTimer() {
  console.log("pause清除了定时器");
  clearInterval(clock.timer);
  clock.title = clock.text[1];
  clock.isPause = true;
}

function continueTimer() {
  clock.timer = setInterval(countdown, 1000);
  if (clock.workTotalTime) {
    clock.title = clock.text[2];
  }
  clock.isStart = true;
  clock.isPause = false;
  clock.isNumberInput = false;
}

function resetTimer() {
  clear();
  clock.isStart = false;
  clock.breakStatus = false;
  clock.isNumberInput = true;
}

function beforeBreak() {
  clock.title = clock.text[3];
  clock.isStart = false;
  clock.breakStatus = true;
}

function breakTimer() {
  clock.title = clock.text[4];
}

function breakStop() {
  clock.title = clock.text[5];
}

function test() {
  clock.minute--;
  console.log(clock);
}

// 倒计时逻辑
function countdown() {
  if (clock.workTotalTime < 1 && clock.timer) {
    clear();
    // TODO 停止了，处理一下
    // if 工作状态到时间了
    if (!clock.breakStatus) {
      beforeBreak();
    } else {
      // if 休息状态到时间了
      clock.title = clock.text[5];
    }
  } else {
    clock.workTotalTime--;
    console.log("时间在减少 :>> ", clock.workTotalTime);
  }
}

// 判断定时器是否存在
function isTimerExist() {
  if (clock.timer) console.log("timer :>> ", clock.timer);
  else console.log("timer没了", clock.timer);
}

// 补零
function pad(time) {
  return (time < 10 ? "0" : "") + time;
}
</script>

<style lang="scss" scoped>
.container {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
  margin: auto;
  .outerBox {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-direction: column;
    margin: auto;
    border: red 2px solid;

    .clockBox {
      height: 300px;
      width: 300px;
      margin: 50px 0 0 0;
      display: flex;
      align-items: center;
      justify-content: center;
      flex-direction: column;
      border: red 2px solid;
      h1 {
        font-size: 40px;
      }
    }

    .buttonBox {
      height: 100px;
      display: flex;
      align-items: center;
      justify-content: center;
      border: red 2px solid;
      margin: 50px 0 0 0;

      .el-button {
        margin: 0 10px;
      }
    }
    .inputBox {
      height: 100px;
      display: flex;
      align-items: center;
      justify-content: center;
      flex-direction: column;
      border: red 2px solid;
      margin: 50px 0 50px 0;
    }
  }
}
</style>
