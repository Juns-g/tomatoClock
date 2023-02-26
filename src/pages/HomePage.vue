<template>
  <div class="container">
    <div class="outerBox">
      <div class="clockBox">
        <h1>{{ clock.title }}</h1>
        <!-- ! 一次学1h及以上是不正确的 -->
        <!-- <h1>{{ clock.hour }}:{{ clock.minute }}:{{ clock.second }}</h1> -->
        <h1>{{ clock.minute }}:{{ clock.second }}</h1>
        <!--  总时间，之后删了 -->
        <!-- <h1>{{ clock.workTotalTime }}</h1> -->
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
          开始专注
        </el-button>
        <el-button
          class="startBreakButton"
          type="warning"
          size="large"
          v-show="!clock.isStart && clock.breakStatus"
          round
          @click="breakTimer()"
        >
          开始休息
        </el-button>
        <el-button
          class="pauseButton"
          type="success"
          size="large"
          v-show="!clock.isPause && clock.isStart && !clock.breakStatus"
          round
          @click="pauseTimer()"
        >
          暂停专注
        </el-button>
        <el-button
          class="continueButton"
          type="success"
          size="large"
          v-show="clock.isPause && clock.isStart && !clock.breakStatus"
          round
          @click="continueTimer()"
        >
          继续专注
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
            size="default"
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
            size="default"
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
import { reactive, watchEffect } from "vue";

let clock = reactive({
  // 展示的时间
  // hour: 0,
  minute: 0,
  second: 0,
  // 输入的时间
  // hourInput: 0,
  // 一次专注只准你多少分钟
  // 测试时间是6s
  // workMinuteInput: 0.1,
  // breakMinuteInput: 0.1,
  workMinuteInput: 25,
  breakMinuteInput: 5,
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
  // isNumberInput: true,
});

function workStart() {
  clock.workTotalTime = clock.workMinuteInput * 60;
  clock.minute = Math.floor(clock.workTotalTime / 60) % 60;
  clock.second = pad(clock.workTotalTime % 60);
  clearInterval(clock.timer);
  //console.log("start清除了定时器");
  clock.timer = setInterval(countdown, 1000);
  if (clock.workTotalTime) {
    clock.title = clock.text[2];
  }
  //console.log("start运行了，总时间是 :>> ", clock.workTotalTime);
  clock.isStart = true;
  // clock.isNumberInput = false;
}

function clear() {
  clearInterval(clock.timer);
  //console.log("clear清除了定时器 :>> ", clock.timer);
  clock.workTotalTime = 0;
  //console.log("总时间清零了 :>> ", clock.workTotalTime);
  clock.title = clock.text[0];
}

function pauseTimer() {
  //console.log("pause清除了定时器");
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
  // clock.isNumberInput = false;
}

function resetTimer() {
  clear();
  clock.isStart = false;
  clock.breakStatus = false;
  // clock.isNumberInput = true;
}

function beforeBreak() {
  clock.title = clock.text[3];
  clock.isStart = false;
  clock.breakStatus = true;
}

function breakTimer() {
  clock.title = clock.text[4];
  clock.breakTotalTime = clock.breakMinuteInput * 60;
  clock.minute = Math.floor(clock.breakTotalTime / 60) % 60;
  clock.second = pad(clock.breakTotalTime % 60);
  clearInterval(clock.timer);
  //console.log("breakStart清除了定时器");
  clock.timer = setInterval(breakCountdown, 1000);
  //console.log("breakStart运行了，总时间是 :>> ", clock.breakTotalTime);
  clock.isStart = true;
}

function breakStop() {
  clock.title = clock.text[5];
  clock.breakStatus = false;
  clock.isStart = false;
}

function test() {
  clock.minute--;
  //console.log(clock);
}

// 工作倒计时逻辑
function countdown() {
  // 专注中
  clock.workTotalTime--;
  const stopWorkWatch = watchEffect(() => {
    clock.minute = Math.floor(clock.workTotalTime / 60) % 60;
    clock.second = pad(clock.workTotalTime % 60);
  });
  //console.log("时间在减少 :>> ", clock.workTotalTime);
  // 专注停止
  if (clock.workTotalTime < 1) {
    clear();
    stopWorkWatch();
    beforeBreak();
  }
}

// 休息倒计时逻辑
function breakCountdown() {
  // 休息中
  clock.breakTotalTime--;
  const stopBreakWatch = watchEffect(() => {
    clock.minute = Math.floor(clock.breakTotalTime / 60) % 60;
    clock.second = pad(clock.breakTotalTime % 60);
  });
  //console.log("时间在减少 :>> ", clock.breakTotalTime);
  // 休息停止
  if (clock.breakTotalTime < 1) {
    clear();
    stopBreakWatch();
    breakStop();
  }
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
