<template>
  <div class="cal">
    <div class="cal-header">
      <button class="prevMonth">&lt;</button>
      <div class="date"><span>{{ year }}</span>年<span>{{ month }}</span>月<span>{{ dateNow }}</span>日</div>
      <button class="nextMonth">&gt;</button>
    </div>
    <div class="cal-body">
      <ul class="cal-body-day">
        <li v-for="day of days" :key="day.num">{{ day.text }}</li>
      </ul>
      <ul class="cal-body-date">
        <li class="every-day" v-for="item of dateline" :key="item.todayStr">{{ item.date }}</li>
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
      ]
    }
  },
  computed: {
    year () {
      return this.date.getFullYear()
    },
    month () {
      return this.date.getMonth()
    },
    dateNow () {
      return this.date.getDate()
    },
    dateline () {
      return this.createDateArr(this.date)
    }
  },
  methods: {
    createDateArr
  }
}

function createDateArr (date) {
  const dateArr = [] // 当前月日期数组

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

  for (let i = 1; i <= maxDays; i++) {
    dateArr.push(
      {
        date: i,
        monthPosition: 0, // 0=>当月 -1=>上月 1=> 正月
        todayStr: `${year}-${month}-${i}`
      }
    )
  }

  // 补齐月份前不足的天数
  const firstDay = new Date(new Date(date.getTime()).setDate(1)).getDay()
  const prevDays = firstDay - 0
  if (prevDays) {
    for (let i = 1; i <= prevDays; i++) {
      let currentDate = new Date(new Date(`${year}-${month}-1`).setDate(1 - i))
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
  const padEndNum = 7 - dateArr.length % 7
  if (padEndNum) {
    for (let i = 1; i <= padEndNum; i++) {
      let currentDate = new Date(new Date(`${year}-${month + 1}-1`).setDate(i))
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
</script>

<style scoped lang="scss">
div {
  min-height: 10px;
}
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
}
.cal-header {
  border: 1px solid deeppink;
  @extend %border-box;
  @extend %flex;
  justify-content: space-between;
  padding: 0 10px;
}
.cal-body {
  border: 1px solid darkgreen;
  @extend %border-box;
  @extend %flex;
  @extend %flex-column;
}
.cal-body-day {
  border: 1px solid darkblue;
  @extend %border-box;
  @extend %flex;
  @extend %clean-ul-li-style;
  justify-content: space-around;
}
.cal-body-date {
  border: 1px solid deepskyblue;
  @extend %border-box;
  @extend %flex;
  @extend %flex-wrap;
  @extend %clean-ul-li-style;
  justify-content: space-around;
}
.every-day {
  @extend %border-box;
  width: calc(100% / 7);
  line-height: 2;
  border: 1px solid skyblue;
}
</style>
