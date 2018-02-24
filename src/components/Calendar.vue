<template>
  <div class="cal">
    <div class="cal-header">
      <button
        class="prev-month"
        @click="chCurrentMonth(-1)">&lt;</button>
      <div
        class="cal-header-title"
        v-if="showTitle">
          <div @click="showTitle = false" class="date-info">
            <span>{{ year }}</span>年<span>{{ month + 1}}</span>月<span>{{ dateNow }}</span>日
          </div>
          <span
            v-show="showSkipBtn"
            @click="skipToToday"
            class="skip-btn"
            title="skip to today">skip to today
          </span>
      </div>
      <div class="date-ctrl" v-else>
        <input
          v-model="ctrlArgs.year"
          type="number"
          min="1900"
          max="2100">年
        <select v-model="ctrlArgs.humanMonth">
          <option
            v-for="i in 12"
            :value="i"
            :key="i" >{{ i }}</option>
        </select>月
        <select v-model="ctrlArgs.date">
          <option
            v-for="i of maxDays"
            :value="i"
            :key="i">{{ i }}</option>
        </select>日
        <button
          @click="chCurrentDate">确定</button>
      </div>
      <button
        class="next-month"
        @click="chCurrentMonth(1)">&gt;</button>
    </div>
    <div class="cal-body">
      <ul class="cal-body-day">
        <li v-for="day of days" :key="day.num">{{ day.text }}</li>
      </ul>
      <ul class="cal-body-date">
        <li
          v-for="item of dateline"
          :class="[
            'every-day',
            {
              'not-this-month': item.monthPosition !== 0,
              'today': today.toDateString() === new Date(item.todayStr).toDateString(),
            }]"
          :title="item.todayStr"
          :key="item.todayStr"
          @click="clickEveryDay(item.todayStr)">
            {{ item.date }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Canlendar',
  props: {
    date: {
      type: Date,
      default () {
        return new Date()
      }
    }
  },
  data () {
    return {
      currentDate: new Date(this.date.getTime()),
      days: [
        {text: '日', num: 0},
        {text: '一', num: 1},
        {text: '二', num: 2},
        {text: '三', num: 3},
        {text: '四', num: 4},
        {text: '五', num: 5},
        {text: '六', num: 6}
      ],
      today: new Date(),
      showTitle: true,
      ctrlArgs: {
        year: 1900,
        humanMonth: 1,
        date: 1
      }
    }
  },
  computed: {
    year () {
      return this.currentDate.getFullYear()
    },
    month () {
      return this.currentDate.getMonth()
    },
    dateNow () {
      return this.currentDate.getDate()
    },
    dateline () {
      return this.createDateArr(this.currentDate)
    },
    maxDays () {
      const ctrlDate = new Date(`${this.ctrlArgs.year}-${this.ctrlArgs.humanMonth}-${this.ctrlArgs.date}`)
      return getMaxDays(ctrlDate)
    },
    showSkipBtn () {
      const today = new Date()
      const year = today.getFullYear()
      const month = today.getMonth()

      return !(
        this.currentDate.getFullYear() === year &&
        this.currentDate.getMonth() === month
      )
    }
  },
  methods: {
    createDateArr,
    chCurrentMonth (next) {
      this.currentDate = new Date(this.currentDate.setMonth(this.month + next))
    },
    chCurrentDate () {
      this.currentDate = new Date(`${this.ctrlArgs.year}-${this.ctrlArgs.humanMonth}-${this.ctrlArgs.date}`)
      this.showTitle = true
    },
    clickEveryDay (dateStr) {
      this.currentDate = new Date(dateStr)
      this.$emit('clickDateItem', dateStr)
    },
    skipToToday () {
      this.currentDate = new Date()
    }
  },
  created () {
    this.ctrlArgs.year = this.currentDate.getFullYear()
    this.ctrlArgs.humanMonth = this.currentDate.getMonth() + 1
    this.ctrlArgs.date = this.currentDate.getDate()
  }
}

function createDateArr (date) {
  const dateArr = [] // 当前月日期数组

  const year = date.getFullYear()
  const month = date.getMonth()

  const maxDays = getMaxDays(date)

  for (let i = 1; i <= maxDays; i++) {
    dateArr.push(
      {
        date: i,
        monthPosition: 0, // 0=>当月 -1=>上月 1=> 正月
        todayStr: `${year}-${month + 1}-${i}`
      }
    )
  }

  // 补齐月份前不足的天数
  const firstDay = new Date(new Date(date.getTime()).setDate(1)).getDay()
  const prevDays = firstDay - 0
  if (prevDays) {
    for (let i = 1; i <= prevDays; i++) {
      let currentDate = new Date(new Date(`${year}-${month + 1}-1`).setDate(1 - i))
      dateArr.unshift(
        {
          date: currentDate.getDate(),
          monthPosition: -1,
          todayStr: `${currentDate.getFullYear()}-${currentDate.getMonth() + 1}-${currentDate.getDate()}`
        }
      )
    }
  }

  // 补齐月份后不足的天数
  const padEndNum = dateArr.length % 7
  if (padEndNum > 0 && padEndNum < 7) {
    for (let i = 1; i <= 7 - padEndNum; i++) {
      let currentDate
      if (month + 1 > 11) {
        currentDate = new Date(new Date(`${year + 1}-1-1`).setDate(i))
      } else {
        currentDate = new Date(new Date(`${year}-${month + 2}-1`).setDate(i))
      }
      dateArr.push(
        {
          date: currentDate.getDate(),
          monthPosition: 1,
          todayStr: `${currentDate.getFullYear()}-${currentDate.getMonth() + 1}-${currentDate.getDate()}`
        }
      )
    }
  }

  return dateArr
}

function isLeapYear (fullYear) {
  return fullYear % 400 === 0 || (fullYear % 100 !== 0 && fullYear % 4 === 0)
}

function getMaxDays (date) {
  const BIG_MONTH = [1, 3, 5, 7, 8, 10, 12]
  const year = date.getFullYear()
  const month = date.getMonth() + 1

  let maxDays
  if (BIG_MONTH.indexOf(month) !== -1) {
    maxDays = 31
  } else if (month !== 2) {
    maxDays = 30
  } else {
    maxDays = isLeapYear(year) ? 29 : 28
  }

  return maxDays
}
</script>

<style scoped lang="scss">
$baseFontSize: 12px;
%border-box {
  box-sizing: border-box;
}
%flex {
  display: flex;
}
%flex-column {
  flex-direction: column;
}
%flex-wrap {
  flex-wrap: wrap;
}
%clean-ul-li-style {
  margin: 0;
  padding: 0;
  list-style: none;
}
.cal {
  border: 1px solid gray;
  @extend %border-box;
  @extend %flex;
  @extend %flex-column;
  max-width: 400px;
  min-width: 360px;
  font-size: $baseFontSize;
  line-height: 2.5;
  padding: 5px 8px;
  border-radius: 4px;
  box-shadow: 3px 3px 4px rgba(0,0,0, .4);
}
.cal-header {
  // border: 1px solid deeppink;
  @extend %border-box;
  @extend %flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 10px;
}
.cal-header-title {
  cursor: pointer;
}
.cal-header-title .date-info {
  display: inline-block;
}
.skip-btn {
  display: inline-block;
  width: 10px;
  height: 10px;
  border-radius: 50%;
  overflow: hidden;
  background-color: #e63771;
}
.prev-month,
.next-month {
  width: 25px;
  height: 25px;
  border: none;
  color: #aaa;
  background-color: none;
  outline: none;
  cursor: pointer;
  &:hover {
    color: #000;
  }
  &:active {
    background-color: rgba(245, 147, 179, 0.54);
  }
}
.cal-body {
  // border: 1px solid darkgreen;
  @extend %border-box;
  @extend %flex;
  @extend %flex-column;
}
.cal-body-day {
  // border: 1px solid darkblue;
  @extend %border-box;
  @extend %flex;
  @extend %clean-ul-li-style;
  justify-content: space-around;
}
.cal-body-date {
  // border: 1px solid deepskyblue;
  @extend %border-box;
  @extend %flex;
  @extend %flex-wrap;
  @extend %clean-ul-li-style;
  justify-content: space-around;
}
.every-day {
  @extend %border-box;
  width: calc(100% / 7 );
  line-height: 3;
  margin-bottom: 5px;
  border-left: 3px solid white;
  border-radius: 5%;
  font-size: $baseFontSize * 1.2;
  cursor: pointer;
}
.every-day:hover {
  background-color: rgba(245, 147, 179, 0.54) !important;
}
.not-this-month {
  background-color: #ddd;
}
.today {
  color: #fff;
  background-color: #e63771 !important;
}
</style>
