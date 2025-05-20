<template>
  <div class="container">
    <div class="section">
      <div class="section-header">
        <h1>收支与薪资分配</h1>
      </div>
      <div class="calculator">
        <div class="panels">
          <!-- 收入面板 -->
          <div class="panel income-panel">
            <h2>收入</h2>
            <div class="form-group">
              <label>类型：</label>
              <select v-model="newIncome.type">
                <option value="团费">团费</option>
                <option value="其他">其他</option>
              </select>
            </div>
            <div class="form-group">
              <label>金额：</label>
              <input type="number" v-model="newIncome.amount" min="0" step="0.01">
            </div>
            <div class="form-group">
              <label>数量：</label>
              <input type="number" v-model="newIncome.quantity" min="1" step="1">
            </div>
            <div class="form-group" v-if="newIncome.type === '其他'">
              <label>备注：</label>
              <input type="text" v-model="newIncome.remark" placeholder="请输入备注">
            </div>
            <button @click="addIncome" class="add-btn income-btn">添加收入</button>
            <div class="records">
              <div v-for="(item, index) in incomeItems" :key="index" class="record-item income">
                <div class="record-info">
                  <div class="record-type">{{ item.type === '其他' ? item.remark : item.type }}</div>
                  <div class="record-details">
                    <span>¥{{ item.amount.toFixed(2) }} × {{ item.quantity }}</span>
                    <span>= ¥{{ (item.amount * item.quantity).toFixed(2) }}</span>
                  </div>
                </div>
                <button @click="deleteIncome(index)" class="delete-btn">删除</button>
              </div>
            </div>
          </div>
          <!-- 支出面板 -->
          <div class="panel expense-panel">
            <h2>支出</h2>
            <div class="form-group">
              <label>类型：</label>
              <select v-model="newExpense.type">
                <option value="住宿">住宿</option>
                <option value="包车">包车</option>
                <option value="领队车补">领队车补</option>
                <option value="公共装备">公共装备</option>
                <option value="保险">保险</option>
                <option value="其他">其他</option>
              </select>
            </div>
            <div class="form-group">
              <label>金额：</label>
              <input type="number" v-model="newExpense.amount" min="0" step="0.01">
            </div>
            <div class="form-group">
              <label>数量：</label>
              <input type="number" v-model="newExpense.quantity" min="1" step="1">
            </div>
            <div class="form-group" v-if="newExpense.type === '其他'">
              <label>备注：</label>
              <input type="text" v-model="newExpense.remark" placeholder="请输入备注">
            </div>
            <button @click="addExpense" class="add-btn expense-btn">添加支出</button>
            <div class="records">
              <div v-for="(item, index) in expenseItems" :key="index" class="record-item expense">
                <div class="record-info">
                  <div class="record-type">{{ item.type === '其他' ? item.remark : item.type }}</div>
                  <div class="record-details">
                    <span>¥{{ item.amount.toFixed(2) }} × {{ item.quantity }}</span>
                    <span>= ¥{{ (item.amount * item.quantity).toFixed(2) }}</span>
                  </div>
                </div>
                <button @click="deleteExpense(index)" class="delete-btn">删除</button>
              </div>
            </div>
          </div>
        </div>
        <div class="summary-section">
          <h2>收支汇总</h2>
          <div class="summary">
            <p class="income-text">总收入：¥{{ totalIncome.toFixed(2) }}</p>
            <p class="expense-text">总支出：¥{{ totalExpense.toFixed(2) }}</p>
            <p class="balance-text">结余（可分配）：¥{{ balance.toFixed(2) }}</p>
          </div>
        </div>
        <div class="form-panel">
          <h2>薪资分配</h2>
          <div class="form-group">
            <label>外来领队人数：</label>
            <select v-model="salaryInfo.externalLeaders">
              <option value="0">0人</option>
              <option value="1">1人</option>
              <option value="2">2人</option>
            </select>
          </div>
          <div v-for="person in persons" :key="person.name" class="form-group">
            <label>{{ person.name }}：</label>
            <div style="display: flex; gap: 10px; flex-wrap: wrap;">
              <label v-for="role in roles" :key="role.value" style="font-weight: normal;">
                <input type="checkbox" v-model="person.selectedRoles" :value="role.value"> {{ role.label }}
              </label>
            </div>
          </div>
        </div>
        <div class="form-panel">
          <h2>分配结果</h2>
          <div v-for="person in persons" :key="person.name" style="margin-bottom: 10px;">
            <strong>{{ person.name }}：</strong> ¥{{ autoSalaryResult[person.name]?.toFixed(2) || '0.00' }}
          </div>
          <div v-if="autoSalaryResult.externalLeaders && autoSalaryResult.externalLeaders.length">
            <div v-for="(amt, idx) in autoSalaryResult.externalLeaders" :key="idx">
              <strong>外来领队{{ idx + 1 }}：</strong> ¥{{ amt.toFixed(2) }}
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

// 当前显示的板块
const currentSection = ref('menu')

const newIncome = ref({
  type: '团费',
  amount: 0,
  quantity: 1,
  remark: ''
})

const newExpense = ref({
  type: '住宿',
  amount: 0,
  quantity: 1,
  remark: ''
})

const incomeItems = ref([])
const expenseItems = ref([])

const addIncome = () => {
  if (newIncome.value.amount <= 0) {
    alert('请输入有效金额')
    return
  }
  if (newIncome.value.quantity <= 0) {
    alert('请输入有效数量')
    return
  }
  if (newIncome.value.type === '其他' && !newIncome.value.remark) {
    alert('请输入备注')
    return
  }
  incomeItems.value.push({...newIncome.value})
  newIncome.value.amount = 0
  newIncome.value.quantity = 1
  newIncome.value.remark = ''
}

const addExpense = () => {
  if (newExpense.value.amount <= 0) {
    alert('请输入有效金额')
    return
  }
  if (newExpense.value.quantity <= 0) {
    alert('请输入有效数量')
    return
  }
  if (newExpense.value.type === '其他' && !newExpense.value.remark) {
    alert('请输入备注')
    return
  }
  expenseItems.value.push({...newExpense.value})
  newExpense.value.amount = 0
  newExpense.value.quantity = 1
  newExpense.value.remark = ''
}

const deleteIncome = (index) => {
  incomeItems.value.splice(index, 1)
}

const deleteExpense = (index) => {
  expenseItems.value.splice(index, 1)
}

const totalIncome = computed(() => {
  return incomeItems.value.reduce((sum, item) => sum + (Number(item.amount) * Number(item.quantity)), 0)
})

const totalExpense = computed(() => {
  return expenseItems.value.reduce((sum, item) => sum + (Number(item.amount) * Number(item.quantity)), 0)
})

const balance = computed(() => {
  return totalIncome.value - totalExpense.value
})

// 薪资分配相关数据
const salaryInfo = ref({
  externalLeaders: 0
})

// 计算可分配金额
const distributableMoney = balance // balance 直接作为可分配金额

const roles = [
  { label: '带线', value: 'guide' },
  { label: '客服', value: 'service' },
  { label: '宣传', value: 'promotion' },
  { label: '路线', value: 'route' },
  { label: '外联', value: 'liaison' }
]

const persons = ref([
  { name: 'A', selectedRoles: [] },
  { name: 'B', selectedRoles: [] },
  { name: 'C', selectedRoles: [] },
  { name: 'D', selectedRoles: [] }
])

const autoSalaryResult = computed(() => {
  const total = balance.value
  if (total <= 0) {
    return { A: 0, B: 0, C: 0, D: 0, externalLeaders: [] }
  }
  const baseRate = 0.08
  const workRates = {
    guide: 0.3,
    service: 0.15,
    promotion: 0.15,
    route: 0.2,
    liaison: 0.2
  }
  const extLeaderNum = Number(salaryInfo.value.externalLeaders)
  let extLeaderRates = []
  if (extLeaderNum === 1) extLeaderRates = [0.2]
  if (extLeaderNum === 2) extLeaderRates = [0.15, 0.15]
  // 1. 基础工资
  const baseSalary = total * baseRate
  // 2. 外来领队分成
  let extLeaderAmounts = extLeaderRates.map(r => total * r)
  let left = total - extLeaderAmounts.reduce((a, b) => a + b, 0)
  // 3. abcd基础工资
  let abcdBase = baseSalary * 4
  left -= abcdBase
  // 4. 分工分配
  let workPool = {}
  Object.keys(workRates).forEach(role => {
    workPool[role] = left > 0 ? left * workRates[role] : 0
  })
  // 统计每个角色有几个人
  let roleCount = {}
  Object.keys(workRates).forEach(role => {
    roleCount[role] = persons.value.filter(p => p.selectedRoles.includes(role)).length
  })
  // 计算每个人的分工分配
  let personWork = {}
  persons.value.forEach(p => {
    let sum = 0
    p.selectedRoles.forEach(role => {
      if (roleCount[role] > 0) {
        sum += workPool[role] / roleCount[role]
      }
    })
    personWork[p.name] = sum
  })
  // 5. 汇总
  let result = {}
  persons.value.forEach(p => {
    result[p.name] = baseSalary + (personWork[p.name] || 0)
  })
  result.externalLeaders = extLeaderAmounts
  return result
})
</script>

<style>
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

h1 {
  text-align: center;
  color: #333;
  margin-bottom: 30px;
}

.calculator {
  background: #f5f5f5;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.panels {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
  margin-bottom: 30px;
}

.panel {
  background: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.income-panel {
  border-top: 4px solid #4CAF50;
}

.expense-panel {
  border-top: 4px solid #f44336;
}

.form-group {
  margin-bottom: 15px;
  display: flex;
  align-items: center;
}

.form-group label {
  width: 80px;
  margin-right: 10px;
}

input, select {
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  flex: 1;
}

.add-btn {
  width: 100%;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
  margin-bottom: 20px;
}

.income-btn {
  background: #4CAF50;
}

.income-btn:hover {
  background: #45a049;
}

.expense-btn {
  background: #f44336;
}

.expense-btn:hover {
  background: #d32f2f;
}

.summary {
  background: white;
  padding: 20px;
  border-radius: 8px;
  margin: 20px 0;
  text-align: center;
}

.summary p {
  font-size: 1.2em;
  margin: 10px 0;
}

.income-text {
  color: #4CAF50;
}

.expense-text {
  color: #f44336;
}

.balance-text {
  font-weight: bold;
  font-size: 1.4em !important;
}

.record-item {
  display: flex;
  justify-content: space-between;
  padding: 10px;
  margin: 5px 0;
  background: #f9f9f9;
  border-radius: 4px;
  align-items: center;
}

.record-info {
  flex: 1;
  margin-right: 10px;
}

.record-type {
  font-weight: bold;
  margin-bottom: 4px;
}

.record-details {
  display: flex;
  gap: 10px;
  color: #666;
  font-size: 0.9em;
}

.record-item.income {
  border-left: 4px solid #4CAF50;
}

.record-item.expense {
  border-left: 4px solid #f44336;
}

.delete-btn {
  background: #ff5252;
  color: white;
  border: none;
  padding: 5px 10px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.8em;
}

.delete-btn:hover {
  background: #d32f2f;
}

.records {
  max-height: 300px;
  overflow-y: auto;
  margin-top: 20px;
}

h2 {
  color: #444;
  margin-bottom: 20px;
  text-align: center;
}

.summary-section {
  margin-top: 30px;
}

.main-menu {
  max-width: 800px;
  margin: 40px auto;
  text-align: center;
}

.menu-buttons {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
  margin-top: 40px;
}

.menu-btn {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 30px;
  border: none;
  border-radius: 10px;
  background: white;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
}

.menu-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 8px rgba(0, 0, 0, 0.15);
}

.finance-btn {
  border-top: 4px solid #4CAF50;
}

.salary-btn {
  border-top: 4px solid #2196F3;
}

.btn-title {
  font-size: 1.5em;
  font-weight: bold;
  margin-bottom: 10px;
}

.btn-desc {
  color: #666;
  font-size: 0.9em;
}

.section-header {
  display: flex;
  align-items: center;
  margin-bottom: 30px;
}

.back-btn {
  background: #666;
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
  margin-right: 20px;
}

.back-btn:hover {
  background: #555;
}

.salary-calculator {
  background: #f5f5f5;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.form-panel {
  background: white;
  padding: 20px;
  border-radius: 8px;
  margin-bottom: 20px;
}

.result-panel {
  background: #f8f9fa;
  padding: 20px;
  border-radius: 8px;
  margin-top: 20px;
  text-align: center;
}

.result-panel h3 {
  color: #333;
  margin-bottom: 10px;
}

.amount {
  font-size: 2em;
  font-weight: bold;
  color: #2196F3;
}
</style> 