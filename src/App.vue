<template>
  <div id="app">
    <section>
      <h2>月次収支</h2>
      <div>
        <table>
          <tr>
            <th>当月収入</th>
            <th></th>
            <th>当月支出</th>
            <th></th>
            <th>当月収支</th>
          </tr>
          <tr>
            <td>{{ currentMonthIncome }}</td>
            <td>-</td>
            <td>{{ currentMonthExpenses }}</td>
            <td>=</td>
            <td>{{ currentMonthIncomeExpenses }}</td>
          </tr>
        </table>

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
              <th>日</th>
              <th>月</th>
              <th>火</th>
              <th>水</th>
              <th>木</th>
              <th>金</th>
              <th>土</th>
            </tr>
            <tbody>
              <tr v-for="(row, i) in calendar" :key="i">
                <td v-for="(col, j) in row" :key="j" :class="{ 'current-month': col.isCurrentMonth  }">
                  <div>{{ col.date.format("MM/DD") }}</div>
                  <div v-if="col.totalIncome !== 0">{{ col.totalIncome }}</div>
                  <div v-if="col.totalExpenses !== 0">{{ col.totalExpenses }}</div>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </section>
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

import _ from 'lodash'
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
    },
    findIncomeByDate (date) {
      return this.incomeExpensesData.filter(item => {
        return item.type === '収入' && dayjs(item.date, 'YYYY/MM/DD').isSame(date)
      })
    },
    findExpensesByDate (date) {
      return this.incomeExpensesData.filter(item => {
        return item.type === '支出' && dayjs(item.date, 'YYYY/MM/DD').isSame(date)
      })
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
    },
    // 表示している期間の合計収入
    currentMonthIncome () {
      let totalIncome = 0
      this.incomeExpensesDetailsData.forEach(item => {
        if (item.type === '収入') {
          totalIncome += item.amount
        }
      })

      return totalIncome
    },
    // 表示している期間の合計支出
    currentMonthExpenses () {
      let totalExpenses = 0
      this.incomeExpensesDetailsData.forEach(item => {
        if (item.type === '支出') {
          totalExpenses += item.amount
        }
      })

      return totalExpenses * -1
    },
    // 表示している期間の収支
    currentMonthIncomeExpenses () {
      return this.currentMonthIncome - this.currentMonthExpenses
    },
    calendar () {
      const startDate = this.displayDateRangeStart.startOf('w')
      const endDate = this.displayDateRangeStart.endOf('M').endOf('w').add(1, 'd')

      const calendar = []

      const days = endDate.diff(startDate, 'd')

      for (var i = 0; i < days; i++) {
        const rowIndex = Math.floor(i / 7)
        const colIndex = i % 7
        if (calendar[rowIndex] === undefined) {
          calendar[rowIndex] = []
        }

        const date = startDate.add(i, 'd')
        calendar[rowIndex][colIndex] = {
          date: date,
          isCurrentMonth: date.month() === this.displayDateRangeStart.month(),
          income: this.findIncomeByDate(date),
          totalIncome: _.sumBy(this.findIncomeByDate(date), 'amount'),
          expenses: this.findExpensesByDate(date),
          totalExpenses: _.sumBy(this.findExpensesByDate(date), 'amount')
        }
      }

      return calendar
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
