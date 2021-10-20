<template>
  <div id="app">
    <section>
      <h2>収入・支出詳細</h2>
      <div>
        <div>
          <button @click="prevMonth">◀︎</button>
          <span>{{ displayDateRange[0].format("YYYY/MM/DD") }}</span>
          <span>-</span>
          <span>{{ displayDateRange[1].format("YYYY/MM/DD") }}</span>
          <button @click="nextMonth">▶︎</button>
          <button @click="nowMonth">今月</button>
        </div>
        <table>
          <tr>
            <th>日付</th>
            <th>内容</th>
            <th>金額（円）</th>
            <th>大項目</th>
            <th>中項目</th>
            <th>メモ</th>
          </tr>
          <tr v-for="row in incomeExpensesDetailsData" :key="row.id">
            <th>{{ row.date }}</th>
            <th>{{ row.content }}</th>
            <th>{{ row.amount }}</th>
            <th>{{ row.largeCategory }}</th>
            <th>{{ row.middleCategory }}</th>
            <th>{{ row.memo }}</th>
          </tr>
        </table>
      </div>
    </section>
  </div>
</template>

<script>
import IncomeExpensesData from './data.json'
import customParseFormat from 'dayjs/plugin/customParseFormat'
import isBetween from 'dayjs/plugin/isBetween'
import dayjs from 'dayjs'
dayjs.extend(customParseFormat)
dayjs.extend(isBetween)

export default {
  name: 'App',
  data () {
    return {
      incomeExpensesData: [...IncomeExpensesData],
      displayDateRangeStart: dayjs(new Date(2021, 9, 1, 0, 0, 0, 0))
    }
  },
  methods: {
    nextMonth () {
      this.displayDateRangeStart = this.displayDateRangeStart.add(1, 'month')
    },
    prevMonth () {
      this.displayDateRangeStart = this.displayDateRangeStart.subtract(1, 'month')
    },
    nowMonth () {
      this.displayDateRangeStart = dayjs().date(1).hour(0).minute(0).second(0).millisecond(0)
    }
  },
  computed: {
    incomeExpensesDetailsData () {
      return [...this.incomeExpensesData].sort((a, b) => {
        return dayjs(b.date, 'YYYY/MM/DD').unix() - dayjs(a.date, 'YYYY/MM/DD').unix()
      }).filter(item => {
        return dayjs(item.date, 'YYYY/MM/DD').isBetween(this.displayDateRange[0], this.displayDateRange[1])
      })
    },
    // 表示期間
    displayDateRange () {
      return [this.displayDateRangeStart, this.displayDateRangeStart.add(1, 'month')]
    }
  }
}
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
