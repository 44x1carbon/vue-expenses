<template>
  <div id="app" class="container">
    <section>
      <h2 class="heading2">月次収支</h2>
      <div>
        <div class="monthly-total">
          <table>
            <thead>
              <tr>
                <th>当月収入</th>
                <th></th>
                <th>当月支出</th>
                <th></th>
                <th>当月収支</th>
              </tr>
            </thead>
            <tbody>
              <tr>
                <td>{{ currentMonthIncome }}</td>
                <td>-</td>
                <td>{{ currentMonthExpenses }}</td>
                <td>=</td>
                <td>{{ currentMonthIncomeExpenses }}</td>
              </tr>
            </tbody>
          </table>
        </div>

        <div>
          <div class="month-controller">
            <button @click="prevMonth" class="month-button">◀︎</button>
            <span class="month-range">
              <span>{{ displayDateRange[0].format("YYYY/MM/DD") }}</span>
              <span>-</span>
              <span>{{ displayDateRange[1].format("YYYY/MM/DD") }}</span>
            </span>
            <button @click="nextMonth" class="month-button">▶︎</button>
            <button @click="nowMonth" class="month-button">今月</button>
          </div>
          <div class="month-calender">
            <table>
              <thead>
                <tr>
                  <th class="sun">日</th>
                  <th>月</th>
                  <th>火</th>
                  <th>水</th>
                  <th>木</th>
                  <th>金</th>
                  <th class="sat">土</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(row, i) in calendar" :key="i">
                  <td
                    v-for="(col, j) in row"
                    :key="j"
                    :class="{ 'current-month': col.isCurrentMonth }"
                  >
                    <div class="day-text">{{ col.date.format("MM/DD") }}</div>
                    <div v-if="col.totalIncome !== 0" class="total-income-text">
                      {{ col.totalIncome }}
                    </div>
                    <div
                      v-if="col.totalExpenses !== 0"
                      class="total-expenses-text"
                    >
                      {{ col.totalExpenses }}
                    </div>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </section>
    <section>
      <h2 class="heading2">収入・支出詳細</h2>
      <div>
        <button @click="isModalShow = true" class="input-button">手入力</button>
        <div class="month-controller">
          <button @click="prevMonth" class="month-button">◀︎</button>
          <span class="month-range">
            <span>{{ displayDateRange[0].format("YYYY/MM/DD") }}</span>
            <span>-</span>
            <span>{{ displayDateRange[1].format("YYYY/MM/DD") }}</span>
          </span>
          <button @click="nextMonth" class="month-button">▶︎</button>
          <button @click="nowMonth" class="month-button">今月</button>
        </div>
        <table class="month-detail">
          <tr>
            <th>日付</th>
            <th>内容</th>
            <th>金額（円）</th>
            <th>大項目</th>
            <th>中項目</th>
            <th>メモ</th>
          </tr>
          <tr v-for="row in incomeExpensesDetailsData" :key="row.id">
            <td>{{ row.date }}</td>
            <td>{{ row.content }}</td>
            <td>{{ row.amount }}</td>
            <td>{{ row.largeCategory }}</td>
            <td>{{ row.middleCategory }}</td>
            <td>{{ row.memo }}</td>
          </tr>
        </table>
      </div>
    </section>
    <div class="modal-outer" v-if="isModalShow">
      <div class="modal-overlay" @click="isModalShow = false"></div>
      <div class="modal">
        <div class="modal-header">
          <h2 class="modal-title">家計簿入力<button class="modal-close" @click="isModalShow = false">×</button></h2>
        </div>
        <div class="modal-contents">
          <div class="tabs">
            <div
              class="tab expenses"
              @click="modalTab = '支出'"
              :class="modalTab == '支出' ? 'active' : ''"
            >
              支出
            </div>
            <div
              class="tab income"
              @click="modalTab = '収入'"
              :class="modalTab == '収入' ? 'active' : ''"
            >
              収入
            </div>
          </div>
          <div v-if="modalTab == '支出'">
            <table class="form-table expenses">
              <tr>
                <th>日付</th>
                <td><input type="date" v-model="expensesInput.date" /></td>
              </tr>
              <tr>
                <th>支出金額</th>
                <td>
                  <div class="input-amount">
                    <span>-</span
                    ><input type="number" v-model="expensesInput.amount" /><span
                      >円</span
                    >
                  </div>
                </td>
              </tr>
              <tr>
                <th>項目</th>
                <td class="input-category">
                  <select v-model="expensesInput.largeCategory">
                    <option
                      v-for="category in Object.keys(expensesCategory)"
                      :key="category"
                      :value="category"
                    >
                      {{ category }}
                    </option>
                  </select>
                  <select v-model="expensesInput.middleCategory">
                    <option
                      v-for="category in expensesCategory[
                        expensesInput.largeCategory
                      ]"
                      :key="category"
                      :value="category"
                    >
                      {{ category }}
                    </option>
                  </select>
                  <input
                    type="text"
                    placeholder="内容をご入力下さい(任意)"
                    v-model="expensesInput.content"
                  />
                </td>
              </tr>
            </table>

            <button @click="saveExpenses" class="save-button">保存する</button>
          </div>
          <div v-if="modalTab == '収入'">
            <table class="form-table income">
              <tr>
                <th>日付</th>
                <td><input type="date" v-model="incomeInput.date" /></td>
              </tr>
              <tr>
                <th>収入金額</th>
                <td>
                  <div class="input-amount">
                    <span>+</span
                    ><input type="number" v-model="incomeInput.amount" /><span
                      >円</span
                    >
                  </div>
                </td>
              </tr>
              <tr>
                <th>項目</th>
                <td class="input-category">
                  <select v-model="incomeInput.largeCategory">
                    <option
                      v-for="category in Object.keys(incomeCategory)"
                      :key="category"
                      :value="category"
                    >
                      {{ category }}
                    </option>
                  </select>
                  <select v-model="incomeInput.middleCategory">
                    <option
                      v-for="category in incomeCategory[
                        incomeInput.largeCategory
                      ]"
                      :key="category"
                      :value="category"
                    >
                      {{ category }}
                    </option>
                  </select>
                  <input
                    type="text"
                    placeholder="内容をご入力下さい(任意)"
                    v-model="incomeInput.content"
                  />
                </td>
              </tr>
            </table>

            <button @click="saveIncome" class="save-button">保存する</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import IncomeExpensesData from './data.json'
import customParseFormat from 'dayjs/plugin/customParseFormat'
import isBetween from 'dayjs/plugin/isBetween'
import dayjs from 'dayjs'
import _ from 'lodash'
import ExpensesCategory from './expenses-category.json'
import IncomeCategory from './income-category.json'
dayjs.extend(customParseFormat)
dayjs.extend(isBetween)

export default {
  name: 'App',
  data () {
    return {
      incomeExpensesData: [...IncomeExpensesData],
      displayDateRangeStart: dayjs(new Date(2021, 9, 1, 0, 0, 0, 0)),
      expensesCategory: ExpensesCategory,
      incomeCategory: IncomeCategory,
      modalTab: '支出',
      isModalShow: false,

      expensesInput: {
        date: dayjs().format('YYYY-MM-DD'),
        amount: 0,
        largeCategory: Object.keys(ExpensesCategory)[0],
        middleCategory: ExpensesCategory[Object.keys(ExpensesCategory)[0]][0],
        content: ''
      },
      incomeInput: {
        date: dayjs().format('YYYY-MM-DD'),
        amount: 0,
        largeCategory: Object.keys(IncomeCategory)[0],
        middleCategory: IncomeCategory[Object.keys(IncomeCategory)[0]][0],
        content: ''
      }
    }
  },
  methods: {
    nextMonth () {
      this.displayDateRangeStart = this.displayDateRangeStart.add(1, 'month')
    },
    prevMonth () {
      this.displayDateRangeStart = this.displayDateRangeStart.subtract(
        1,
        'month'
      )
    },
    nowMonth () {
      this.displayDateRangeStart = dayjs()
        .date(1)
        .hour(0)
        .minute(0)
        .second(0)
        .millisecond(0)
    },
    findIncomeByDate (date) {
      return this.incomeExpensesData.filter((item) => {
        return (
          item.type === '収入' && dayjs(item.date, 'YYYY/MM/DD').isSame(date)
        )
      })
    },
    findExpensesByDate (date) {
      return this.incomeExpensesData.filter((item) => {
        return (
          item.type === '支出' && dayjs(item.date, 'YYYY/MM/DD').isSame(date)
        )
      })
    },
    saveIncome () {
      this.incomeExpensesData.push({
        id: '',
        type: '収入',
        amount: this.incomeInput.amount,
        date: dayjs(this.incomeInput.date, 'YYYY/MM/DD'),
        content: this.incomeInput.content,
        largeCategory: this.incomeInput.largeCategory,
        middleCategory: this.incomeInput.middleCategory
      })
    },
    saveExpenses () {
      this.incomeExpensesData.push({
        id: '',
        type: '支出',
        amount: this.expensesInput.amount * -1,
        date: dayjs(this.expensesInput.date, 'YYYY/MM/DD'),
        content: this.expensesInput.content,
        largeCategory: this.expensesInput.largeCategory,
        middleCategory: this.expensesInput.middleCategory
      })
    }
  },
  computed: {
    incomeExpensesDetailsData () {
      return [...this.incomeExpensesData]
        .sort((a, b) => {
          return (
            dayjs(b.date, 'YYYY/MM/DD').unix() -
            dayjs(a.date, 'YYYY/MM/DD').unix()
          )
        })
        .filter((item) => {
          return dayjs(item.date, 'YYYY/MM/DD').isBetween(
            this.displayDateRange[0],
            this.displayDateRange[1]
          )
        })
    },
    // 表示期間
    displayDateRange () {
      return [
        this.displayDateRangeStart,
        this.displayDateRangeStart.add(1, 'month')
      ]
    },
    // 表示している期間の合計収入
    currentMonthIncome () {
      let totalIncome = 0
      this.incomeExpensesDetailsData.forEach((item) => {
        if (item.type === '収入') {
          totalIncome += item.amount
        }
      })

      return totalIncome
    },
    // 表示している期間の合計支出
    currentMonthExpenses () {
      let totalExpenses = 0
      this.incomeExpensesDetailsData.forEach((item) => {
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
      const endDate = this.displayDateRangeStart
        .endOf('M')
        .endOf('w')
        .add(1, 'd')

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
body {
  font-size: 13px;
}

.heading2 {
  font-size: 125%;
  font-weight: bold;
  line-height: 170%;
  border-bottom: 2px solid #e2750f;
  margin: 20px 0 15px;
}

.monthly-total table {
  border: 1px #ddd solid;
  width: 100%;
  font-size: 15px;
  text-align: center;
  max-width: 100%;
  background-color: transparent;
  border-collapse: collapse;
  border-spacing: 0;
}

.monthly-total table th {
  background-color: #faf6f0;
}

.month-controller {
  margin: 20px auto;
  width: fit-content;
}

.month-range {
  text-align: center;
  font-size: 28px;
  position: relative;
  width: 407px;
  vertical-align: middle;
}

.month-button {
  height: 1.9em;
  line-height: 1.9em;
  padding: 0 0.6em;
  font-weight: bold;
  display: inline-block;
  vertical-align: middle;
}

.sun {
  color: red;
}

.sat {
  color: blue;
}

.month-calender {
  margin: 0 auto;
  width: fit-content;
}

.month-calender th {
  text-align: center;
}

.month-calender td {
  text-align: right;
  height: 86px;
}

.month-calender table {
  border-spacing: 0px;
}

.month-calender td:nth-child(1) .day-text {
  color: red;
}

.month-calender td:nth-child(7) .day-text {
  color: blue;
}

.month-calender td:not(.current-month) {
  background-color: #dddddd;
}

.month-calender td:not(.current-month) .day-text {
  opacity: 0.3;
}

.total-income-text {
  color: #4572a7;
}

.total-expenses-text {
  color: #aa4643;
}

.day-text {
  padding: 0 2px;
  font-size: 16px;
  margin: 3px;
}

.month-calender th,
.month-calender td {
  width: 93px;
  border: 1px solid #ccc;
  vertical-align: top;
}

.month-detail {
  border: 1px solid #eeeeee;
  width: 100%;
  border-spacing: 0;
}
.month-detail th {
  background-color: #f9f6f0;
  white-space: nowrap;
  text-align: center;
  border: 1px solid #eeeeee;
  border-right: none;
  padding: 5px;
}

.month-detail td {
  background-color: #ffffff;
  border-collapse: collapse;
  border: 1px solid #eeeeee;
  padding: 5px;
}

.container {
  width: 800px;
  margin: 0 auto;
  margin-bottom: 100px;
}

.input-button {
  background: #ef7f1a;
  padding: 5px 20px;
  width: 140px;
  color: #fff;
  text-shadow: 0 -1px 0 rgb(0 0 0 / 25%);
  border-color: rgba(0, 0, 0, 0.1) rgba(0, 0, 0, 0.1) rgba(0, 0, 0, 0.25);
  font-size: 13px;
  line-height: 18px;
  text-align: center;
  border-radius: 4px;
  margin-left: auto;
  display: block;
  cursor: pointer;
}

.modal-overlay {
  position: fixed;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  z-index: 1040;
  background-color: #000;
  width: 100vw;
  height: 100vh;
  opacity: 0.8;
}

.modal {
  position: fixed;
  width: 500px;
  height: fit-content;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  z-index: 1050;
  background-color: #fff;
  margin: auto;
  border: 1px solid rgba(0, 0, 0, 0.3);
  border-radius: 6px;
  box-shadow: 0 3px 7px rgb(0 0 0 / 30%);
}

.modal-header {
  padding: 9px 15px;
  border-bottom: 1px solid #eee;
}

.modal-contents {
  padding: 15px;
}

.tabs {
  display: flex;
}

.tab {
  flex: 1;
  text-align: center;
  font-size: 15px;
  padding: 20px 0;
  cursor: pointer;
  border-bottom: none;
  border-bottom: 1px solid #e5e5e5;
}

.tab:not(:last-child) {
  border-right: 1px solid #e5e5e5;
}

.tab.active {
  border-bottom: none;
}

.form-table {
  border-top: 1px solid;
  width: 100%;
  border-spacing: 0;
  margin: 20px 0;
}

.form-table th {
  border-right: solid 1px #eee !important;
}

.form-table th,
.form-table td {
  border-bottom: 1px solid;
  padding: 15px;
}

.expenses.form-table {
  border-color: #f0c0bf;
}

.expenses.form-table th,
.expenses.form-table td {
  border-color: #f0c0bf;
}

.expenses.form-table th {
  color: #cb5552;
  font-weight: normal;
}

.income.form-table {
  border-color: #b0d2e4;
}

.income.form-table th,
.income.form-table td {
  border-color: #b0d2e4;
}

.income.form-table th {
  color: #3185b0;
  font-weight: normal;
}

.tab.income {
  color: #3185b0;
}

.tab.income::before {
  content: "+";
  color: #fff;
  font-weight: bold;
  background-color: #3185b0;
  width: 20px;
  height: 20px;
  line-height: 20px;
  display: inline-block;
  border-radius: 50%;
  margin-right: 5px;
}

.tab.expenses {
  color: #cb5552;
}

.tab.expenses::before {
  content: "-";
  color: #fff;
  font-weight: bold;
  background-color: #ca5452;
  width: 20px;
  height: 20px;
  line-height: 20px;
  display: inline-block;
  border-radius: 50%;
  margin-right: 5px;
}

.input-amount span:nth-of-type(1) {
  border-radius: 4px 0 0 4px;
  display: inline-block;
  width: auto;
  height: 18px;
  min-width: 16px;
  padding: 4px 5px;
  font-size: 13px;
  font-weight: normal;
  line-height: 18px;
  text-align: center;
  text-shadow: 0 1px 0 #fff;
  background-color: #eee;
  border: 1px solid #ccc;
}

.input-amount span:nth-of-type(2) {
  border-radius: 0 4px 4px 0;
  display: inline-block;
  width: auto;
  height: 18px;
  min-width: 16px;
  padding: 4px 5px;
  font-size: 13px;
  font-weight: normal;
  line-height: 18px;
  text-align: center;
  text-shadow: 0 1px 0 #fff;
  background-color: #eee;
  border: 1px solid #ccc;
}

.input-amount input {
  height: 18px;
  padding: 4px 6px;
  line-height: 18px;
  color: #555;
  border: 1px solid #ccc;
}

.input-category > * {
  display: block;
}

.input-category > *:not(:last-child) {
  margin-bottom: 10px;
}

.save-button {
  background: #4fbb4f;
  cursor: pointer;
  width: 120px;
  height: 33px;
  display: block;
  margin: 0 auto;
  color: #fff;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.modal-title {
  display: flex;
  justify-content: space-between;
}

.modal-close {
  display: inline-block;
  border: none;
  background-color: transparent;
  font-weight: bold;
  font-size: 17px;
  color: #999;
}
</style>
