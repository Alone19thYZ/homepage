<template>
  <div class="time-capsule">
    <div class="title">
      <hourglass-full theme="two-tone" size="24" :fill="['#efefef', '#00000020']" />
      <span>时光胶囊</span>
      <span class="text">
        <p id="htmer_time"></p>
      </span>
    </div>
    <div v-if="timeData" class="all-capsule">
      <div v-for="(item, tag, index) in timeData" :key="index" class="capsule-item">
        <div class="item-title">
          <span class="percentage">
            {{ item.name }}已度过
            <strong>{{ item.passed }}</strong>
            {{ tag === "day" ? "小时" : "天" }}
          </span>
          <span class="remaining">
            剩余&nbsp;{{ item.remaining }}&nbsp;{{ tag === "day" ? "小时" : "天" }}
          </span>
        </div>
        <el-progress :text-inside="true" :stroke-width="20" :percentage="item.percentage" />
      </div>
      <!-- 建站日期 -->
      <div v-if="store.siteStartShow" class="capsule-item start">
        <div class="item-title">{{ startDateText }}</div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { HourglassFull } from "@icon-park/vue-next";
import { getTimeCapsule, siteDateStatistics } from "@/utils/getTime.js";
import { mainStore } from "@/store";
const store = mainStore();

// 进度条数据
const timeData = ref(getTimeCapsule());
const startDate = ref(import.meta.env.VITE_SITE_START);
const startDateText = ref(null);
const timeInterval = ref(null);

onMounted(() => {
  timeInterval.value = setInterval(() => {
    timeData.value = getTimeCapsule();
    if (startDate.value) startDateText.value = siteDateStatistics(new Date(startDate.value));
  }, 1000);
});

onBeforeUnmount(() => {
  clearInterval(timeInterval.value);
});

// 站点存活时间
let runTime = {
  year: import.meta.env.VITE_SITE_YAER,
  month: import.meta.env.VITE_SITE_MONTH,
  day: import.meta.env.VITE_SITE_DAY,
}

function secondToDate(second) {
  if (!second) {
      return 0;
  }
  var time = new Array(0, 0, 0, 0, 0);
  if (second >= 365 * 24 * 3600) {
      time[0] = parseInt(second / (365 * 24 * 3600));
      second %= 365 * 24 * 3600;
  }
  if (second >= 24 * 3600) {
      time[1] = parseInt(second / (24 * 3600));
      second %= 24 * 3600;
  }
  if (second >= 3600) {
      time[2] = parseInt(second / 3600);
      second %= 3600;
  }
  if (second >= 60) {
      time[3] = parseInt(second / 60);
      second %= 60;
  }
  if (second > 0) {
      time[4] = second;
  }
  return time;
}

function setTime() {
  //month要少一个月，不然会出问题。即month的范围为 0-11
  var create_time = Math.round(new Date(Date.UTC(runTime.year, runTime.month,runTime.day, 0, 0, 0)).getTime() / 1000);
  var timestamp = Math.round((new Date().getTime() + 8 * 60 * 60 * 1000) / 1000);
  var currentTime = secondToDate((  timestamp-create_time));
  var currentTimeHtml = currentTime[0] + ' 年 ' + currentTime[1] + ' 天 '
      + currentTime[2] + ' 时 ' + currentTime[3] + ' 分 ' + currentTime[4]
      + ' 秒';
  if(document.getElementById("htmer_time")!=null)
      document.getElementById("htmer_time").innerHTML = "本站已经苟活 "+currentTimeHtml;
}
// 即时刷新站点存活时间
setInterval(setTime, 1000);


</script>

<style lang="scss" scoped>
.time-capsule {
  width: 100%;
  .title {
    display: flex;
    flex-direction: row;
    align-items: center;
    margin: 0.2rem 0 1.5rem;
    font-size: 1.1rem;
    .i-icon {
      display: flex;
      justify-content: center;
      align-items: center;
      margin-right: 6px;
    }
  }
  .all-capsule {
    .capsule-item {
      margin-bottom: 1rem;
      .item-title {
        display: flex;
        flex-direction: row;
        align-items: center;
        justify-content: space-between;
        margin: 1rem 0rem 0.5rem 0rem;
        font-size: 0.95rem;
        .remaining {
          opacity: 0.6;
          font-size: 0.85rem;
          font-style: oblique;
        }
      }
      &:last-child {
        margin-bottom: 0;
      }
      &.start {
        .item-title {
          justify-content: center;
          opacity: 0.8;
          font-size: 0.85rem;
        }
      }
    }
  }
}
</style>
