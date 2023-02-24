<template>
  <div class="container">
    <div class="outerBox">
      <div class="clockBox">
        <h1>{{ clock.title }}</h1>
        <!-- ! 一次学1h及以上是不正确的 -->
        <!-- <h1>{{ clock.hour }}:{{ clock.minute }}:{{ clock.second }}</h1> -->
        <h1>{{ clock.minute }}:{{ clock.second }}</h1>
        <h1>{{ clock.totalTime }}</h1>
      </div>
      <div class="buttonBox">
        <el-button type="success" size="large" round @click="start()">
          开 始
        </el-button>
        <el-button type="success" size="large" round> 暂 停 </el-button>
        <el-button type="success" size="large" round> 继 续 </el-button>
        <el-button type="success" size="large" round @click="clear()">
          重 置
        </el-button>
        <el-button type="warning" size="large" round @click="test()">
          测试
        </el-button>

        <n-button>naive-ui</n-button>
      </div>
      <div class="inputBox">
        <!-- ! 一次学1h及以上是不正确的 -->
        <!-- <el-input-number
          v-model="clock.hourInput"
          size="large"
          :min="0"
          :max="24"
        /> -->
        <el-input-number
          v-model="clock.minuteInput"
          size="large"
          :min="0"
          :max="60"
        />
        <el-input-number
          v-model="clock.secondInput"
          size="large"
          :min="0"
          :max="60"
        />
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
  hourInput: 0,
  minuteInput: 0,
  secondInput: 0,
  totalTime: 1,
  title: "开始专注吧",
});

// 根据输入的时分秒计算总时间
watch(
  [() => clock.hourInput, () => clock.minuteInput, () => clock.secondInput],
  (newValue, oldValue) => {
    // console.log("oldValue :>> ", oldValue);
    // console.log("newValue :>> ", newValue);
    clock.totalTime =
      clock.hourInput * 3600 + clock.minuteInput * 60 + clock.secondInput;
  }
);

// 根据总时间计算出显示的时间
watch(
  () => clock.totalTime,
  (newValue, oldValue) => {
    // console.log("oldValue :>> ", oldValue);
    // console.log("newValue :>> ", newValue);
    clock.hour = Math.floor(clock.totalTime / 3600);
    clock.minute = Math.floor(clock.totalTime / 60) % 60;
    clock.second = pad(clock.totalTime % 60);
  }
);

// 使用如下的计算属性会导致一开始不显示数字，还不知道why
/* clock.hour = computed(() => {
  Math.floor(clock.totalTime / 3600);
});
clock.minute = computed(() => {
  Math.floor(clock.totalTime / 60) % 60;
});
clock.second = computed(() => {
  clock.totalTime % 60;
}); */

// 时分秒 转 秒
function time2TotalTime(hour, minute, second) {
  return hour * 3600 + minute * 60 + second;
}

// 秒 转 时分秒
function totalTime2time(totalTime) {
  let hour = Math.floor(totalTime / 3600);
  let minute = Math.floor(totalTime / 60) % 60;
  let second = totalTime % 60;
  clock.hour = hour;
  clock.minute = minute;
  clock.second = second;
}

function start() {
  clear();
  //   var timeStart =
  const timer = setInterval(() => countdown(), 1000);
  // const timer = setInterval(countdown(), 1000);
  //   timeStart;
  console.log("clock.totalTime :>> ", clock.totalTime);
}

function clear() {
  //   clearInterval(timeStart);
  console.log("清除了倒计时");
}

function test() {
  clock.minute--;
  clock.totalTime = time2TotalTime(clock.hour, clock.minute, clock.second);
  console.log(clock);
}

// 补零
function pad(time) {
  return (time < 10 ? "0" : "") + time;
}

// 倒计时逻辑
function countdown() {
  if (clock.totalTime < 1) {
    clearInterval(timer);
    console.log("总时间清零了 :>> ", clock.totalTime);

    // TODO 停止了，处理一下
    // if 休息状态到时间了
    // if 工作状态到时间了
  }
  clock.totalTime--;
}
</script>

<style lang="scss" scoped>
.container {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
  .outerBox {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-direction: column;
    border: red 2px solid;

    .clockBox {
      height: 300px;
      width: 300px;
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
    }
    .inputBox {
      height: 100px;
      display: flex;
      align-items: center;
      justify-content: center;
      flex-direction: column;
      border: red 2px solid;
      margin: 100px 0;
    }
  }
}
</style>
