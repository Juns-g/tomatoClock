<template>
  <div class="container">
    <div class="outerBox">
      <div class="clockBox">
        <div class="clockBox-title">
          <h1>{{ clock.title }}</h1>
        </div>
        <!-- ! 一次学1h及以上是不正确的 -->
        <!-- <h1>{{ clock.hour }}:{{ clock.minute }}:{{ clock.second }}</h1> -->
        <n-progress
          type="circle"
          :percentage="clock.timePercentage"
          :color="colors.orange"
          :rail-color="colors.orangeOpacity"
          :offset-degree="180"
          v-if="clock.isOneCircle"
          class="clockBox-timeProgress"
        >
          <div class="clockBox-timeProgress-time">
            <h1>{{ clock.minute }}:{{ clock.second }}</h1>
          </div>
        </n-progress>
        <n-progress
          type="multiple-circle"
          :percentage="[clock.minutePercentage, clock.secondPercentage]"
          :color="[colors.orange, colors.lightgreen]"
          :rail-style="[
            { stroke: colors.orange, opacity: 0.3 },
            { stroke: colors.lightgreen, opacity: 0.3 },
          ]"
          v-if="!clock.isOneCircle"
          class="clockBox-timeProgress"
        >
          <div class="clockBox-timeProgress-time">
            <h1>{{ clock.minute }}:{{ clock.second }}</h1>
          </div>
        </n-progress>

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
          v-show="!clock.isPause && clock.isStart && !clock.breakStatus"
          round
          @click="pauseTimer()"
        >
          暂 停
        </el-button>
        <el-button
          class="continueButton"
          type="success"
          size="large"
          v-show="clock.isPause && clock.isStart && !clock.breakStatus"
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
      </div>
      <div class="settingsBox">
        <!-- ! 一次学1h及以上是不正确的 -->
        <!-- <el-input-number
          v-model="clock.hourInput"
          size="large"
          :min="0"
          :max="24"
        /> -->
        <div class="workTimeInput timeInput">
          <span>专注时间: </span>
          <el-input-number
            v-model="clock.workMinuteInput"
            size="default"
            :min="0"
            :max="60"
          />
          <span> min </span>
        </div>
        <div class="breakTimeInput timeInput">
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
        <div class="settingsButtons">
          <el-button
            type="success"
            plain
            class="settingButton"
            @click="clock.isOneCircle = !clock.isOneCircle"
            >切换圆环</el-button
          >
          <el-button
            type="success"
            plain
            class="settingButton"
            @click="aboutClick()"
            >关于作者</el-button
          >
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { reactive } from "vue";
import { NProgress } from "naive-ui";

let colors = {
  red: "#ff3000",
  orange: "#ff601a",
  orangeOpacity: "rgba(255, 96, 26,0.3)",
  darkgreen: "#167203",
  lightgreen: "#5da500",
  purple: "#6f42c1",
};

let clock = reactive({
  // 展示的时间
  // hour: 0,
  minute: 0,
  second: 0,
  timePercentage: 0,
  minutePercentage: 0,
  secondPercentage: 0,
  // 输入的时间
  // hourInput: 0,
  // 一次专注只准你多少分钟
  // 测试时间是6s
  workMinuteInput: 0.1,
  breakMinuteInput: 0.1,
  // 下面是正式的默认时间
  // workMinuteInput: 25,
  // breakMinuteInput: 5,
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
  isOneCircle: true,
  // isNumberInput: true,
});

function workStart() {
  clock.workTotalTime = clock.workMinuteInput * 60;
  changeDisplay(clock.workTotalTime, clock.workMinuteInput);

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
  clock.timePercentage = 0;
  clock.secondPercentage = 0;
  clock.minutePercentage = 0;
  clock.minute = 0;
  clock.second = 0;
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
  changeDisplay(clock.breakTotalTime, clock.breakMinuteInput);
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

// 工作倒计时逻辑
function countdown() {
  // 专注中
  clock.workTotalTime--;
  // const stopWorkWatch = watchEffect(() => {
  changeDisplay(clock.workTotalTime, clock.workMinuteInput);

  // console.log("时间在减少 :>> ", clock.workTotalTime);
  // 专注停止
  if (clock.workTotalTime == 0) {
    clear();
    // stopWorkWatch();
    beforeBreak();
  }
}

// 休息倒计时逻辑
function breakCountdown() {
  // 休息中
  clock.breakTotalTime--;
  changeDisplay(clock.breakTotalTime, clock.breakMinuteInput);
  // 休息停止
  if (clock.breakTotalTime == 0) {
    clear();
    // stopBreakWatch();
    breakStop();
  }
}

// 补零
function pad(time) {
  return (time < 10 ? "0" : "") + time;
}

// 封装改变显示的函数
function changeDisplay(totalTime, inputTime) {
  clock.minute = Math.floor(totalTime / 60) % 60;
  clock.second = pad(totalTime % 60);
  clock.timePercentage = (totalTime / (inputTime * 60)) * 100;
  clock.minutePercentage = (clock.minute / inputTime) * 100;
  clock.secondPercentage = (clock.second / 60) * 100;
}

function aboutClick() {
  // location.href = "https://github.com/L-H-X";
  window.open("https://github.com/L-H-X");
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
    //border: red 2px solid;

    .clockBox {
      // height: 300px;
      width: 400px;
      margin: 50px 0 0 0;
      display: flex;
      align-items: center;
      justify-content: center;
      flex-direction: column;
      //border: red 2px solid;

      &-title {
        font-size: 20px;
      }

      &-timeProgress {
        margin: 20px 0 0 0;
        width: 60%;
        height: 60%;

        &-time {
          font-size: 20px;
        }
      }
    }

    .buttonBox {
      height: 100px;
      display: flex;
      align-items: center;
      justify-content: center;
      //border: red 2px solid;
      margin: 50px 0 0 0;

      .el-button {
        margin: 0 10px;
      }
    }
    .settingsBox {
      height: 100px;
      display: flex;
      align-items: center;
      justify-content: center;
      flex-direction: column;
      //border: red 2px solid;
      margin: 50px 0 50px 0;

      .timeInput {
        margin: 10px 0;
        font-size: 16px;
      }

      .settingsButtons {
        margin: 5px 0;

        .settingButton {
          margin: 5px 40px;
        }
      }
    }
  }
}
</style>
