<template>
  <div class="dashboard">
    <h1>Welcome back, {{ authStore.user?.name }}!</h1>
    
    <div class="dashboard-grid">
      <div class="card">
        <h3>Total Balance</h3>
        <p class="balance">${{ totalBalance.toFixed(2) }}</p>
      </div>
      
      <div class="card">
        <h3>This Month's Income</h3>
        <p class="income">+${{ monthlyIncome.toFixed(2) }}</p>
      </div>
      
      <div class="card">
        <h3>This Month's Expenses</h3>
        <p class="expense">-${{ monthlyExpenses.toFixed(2) }}</p>
      </div>
      
      <div class="card">
        <h3>Active Budgets</h3>
        <p class="count">{{ activeBudgets.length }}</p>
      </div>
    </div>

    <div class="recent-section">
      <div class="card">
        <h3>Recent Transactions</h3>
        <div v-if="recentTransactions.length === 0" class="empty-state">
          No transactions yet. <router-link to="/transactions">Add your first transaction</router-link>
        </div>
        <div v-else class="transaction-list">
          <div 
            v-for="transaction in recentTransactions.slice(0, 5)" 
            :key="transaction._id"
            class="transaction-item"
          >
            <div class="transaction-info">
              <strong>{{ transaction.description }}</strong>
              <span class="category">{{ transaction.category }}</span>
            </div>
            <div class="transaction-amount" :class="transaction.type">
              {{ transaction.type === 'income' ? '+' : '-' }}${{ transaction.amount.toFixed(2) }}
            </div>
          </div>
          <router-link to="/transactions" class="view-all-link">View all transactions →</router-link>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'
import { useAuthStore } from '../stores/auth'
import api from '../services/api'

export default {
  name: 'Dashboard',
  setup() {
    const authStore = useAuthStore()
    const transactions = ref([])
    const budgets = ref([])

    const totalBalance = computed(() => {
      return transactions.value.reduce((total, transaction) => {
        return transaction.type === 'income' 
          ? total + transaction.amount 
          : total - transaction.amount
      }, 0)
    })

    const monthlyIncome = computed(() => {
      const currentMonth = new Date().getMonth()
      const currentYear = new Date().getFullYear()
      
      return transactions.value
        .filter(t => {
          const date = new Date(t.date)
          return t.type === 'income' && 
                 date.getMonth() === currentMonth && 
                 date.getFullYear() === currentYear
        })
        .reduce((total, transaction) => total + transaction.amount, 0)
    })

    const monthlyExpenses = computed(() => {
      const currentMonth = new Date().getMonth()
      const currentYear = new Date().getFullYear()
      
      return transactions.value
        .filter(t => {
          const date = new Date(t.date)
          return t.type === 'expense' && 
                 date.getMonth() === currentMonth && 
                 date.getFullYear() === currentYear
        })
        .reduce((total, transaction) => total + transaction.amount, 0)
    })

    const recentTransactions = computed(() => {
      return transactions.value
        .sort((a, b) => new Date(b.date) - new Date(a.date))
    })

    const activeBudgets = computed(() => {
      return budgets.value.filter(budget => budget.isActive)
    })

    const loadData = async () => {
      try {
        const [transactionsRes, budgetsRes] = await Promise.all([
          api.get('/transactions'),
          api.get('/budgets')
        ])
        
        transactions.value = transactionsRes.data.transactions
        budgets.value = budgetsRes.data.budgets
      } catch (error) {
        console.error('Error loading dashboard data:', error)
      }
    }

    onMounted(loadData)

    return {
      authStore,
      totalBalance,
      monthlyIncome,
      monthlyExpenses,
      recentTransactions,
      activeBudgets
    }
  }
}
</script>

<style scoped>
.dashboard h1 {
  margin-bottom: 2rem;
  color: #2c3e50;
}

.dashboard-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1rem;
  margin-bottom: 2rem;
}

.balance {
  font-size: 2rem;
  font-weight: bold;
  color: #2c3e50;
  margin: 0;
}

.income {
  font-size: 1.5rem;
  font-weight: bold;
  color: #27ae60;
  margin: 0;
}

.expense {
  font-size: 1.5rem;
  font-weight: bold;
  color: #e74c3c;
  margin: 0;
}

.count {
  font-size: 1.5rem;
  font-weight: bold;
  color: #3498db;
  margin: 0;
}

.transaction-list {
  space: 1rem 0;
}

.transaction-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.5rem 0;
  border-bottom: 1px solid #eee;
}

.transaction-item:last-child {
  border-bottom: none;
}

.transaction-info {
  display: flex;
  flex-direction: column;
}

.category {
  font-size: 0.9rem;
  color: #666;
}

.transaction-amount.income {
  color: #27ae60;
  font-weight: bold;
}

.transaction-amount.expense {
  color: #e74c3c;
  font-weight: bold;
}

.view-all-link {
  display: inline-block;
  margin-top: 1rem;
  color: #3498db;
  text-decoration: none;
}

.view-all-link:hover {
  text-decoration: underline;
}

.empty-state {
  text-align: center;
  color: #666;
  padding: 2rem 0;
}

.empty-state a {
  color: #3498db;
  text-decoration: none;
}

.empty-state a:hover {
  text-decoration: underline;
}
</style>