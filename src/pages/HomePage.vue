<template>
  <div class="container">
    <div class="outerBox">
      <div class="clockBox">
        <h1>{{ clock.hour }}:{{ clock.minute }}:{{ clock.second }}</h1>
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
        <el-button
          type="warning"
          size="large"
          round
          @click="time2TotalTime(clock.hour, clock.minute, clock.second)"
        >
          时分秒 转 秒
        </el-button>
        <el-button
          type="warning"
          size="large"
          round
          @click="totalTime2time(clock.totalTime)"
        >
          秒 转 时分秒
        </el-button>
      </div>
      <div class="inputBox">
        <el-input-number v-model="clock.hour" size="large" :min="0" :max="24" />
        <el-input-number
          v-model="clock.minute"
          size="large"
          :min="0"
          :max="60"
        />
        <el-input-number
          v-model="clock.second"
          size="large"
          :min="0"
          :max="60"
        />
        <el-input-number v-model="clock.totalTime" size="large" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, reactive } from "vue";

let clock = reactive({
  hour: 0,
  minute: 25,
  second: 0,
  totalTime: 0,
});

clock.totalTime = computed(() => {
  return clock.hour * 3600 + clock.minute * 60 + clock.second;
});

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
  var timeStart = setInterval(() => {
    if (clock.totalTime > 0) {
      clock.totalTime--;
    } else {
      clearInterval(timeStart);
    }
    console.log("clock.totalTime :>> ", clock.totalTime);
  }, 1000);
}

function clear() {
  clearInterval(timeStart);
  console.log("清除了倒计时");
}

function test() {
  clock.minute--;
  clock.totalTime = time2TotalTime(clock.hour, clock.minute, clock.second);
  console.log(clock);
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
