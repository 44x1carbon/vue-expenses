<template>
  <div id="app">
    <section>
      <h2>収入・支出詳細</h2>
      <table>
        <tr>
          <th>日付</th>
          <th>内容</th>
          <th>金額（円）</th>
          <th>大項目</th>
          <th>中項目</th>
          <th>メモ</th>
        </tr>
        <tr v-for="row in incomeExpensesData" :key="row.id">
          <th>{{ row.date }}</th>
          <th>{{ row.content }}</th>
          <th>{{ row.amount }}</th>
          <th>{{ row.largeCategory }}</th>
          <th>{{ row.middleCategory }}</th>
          <th>{{ row.memo }}</th>
        </tr>
      </table>
    </section>
  </div>
</template>

<script>
import IncomeExpensesData from './data.json'

export default {
  name: 'App',
  data () {
    return {
      incomeExpensesData: [...IncomeExpensesData],
    }
  },
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
