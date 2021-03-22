<template>
  <div class="transactions">
    <div class="page-header">
      <h1>Transactions</h1>
      <button @click="showAddForm = !showAddForm" class="btn btn-primary">
        {{ showAddForm ? 'Cancel' : 'Add Transaction' }}
      </button>
    </div>

    <div v-if="showAddForm" class="card add-form">
      <h3>Add New Transaction</h3>
      <form @submit.prevent="addTransaction">
        <div class="form-row">
          <div class="form-group">
            <label class="form-label">Description</label>
            <input 
              v-model="newTransaction.description" 
              type="text" 
              class="form-input" 
              required 
            />
          </div>
          <div class="form-group">
            <label class="form-label">Amount</label>
            <input 
              v-model.number="newTransaction.amount" 
              type="number" 
              step="0.01"
              class="form-input" 
              required 
            />
          </div>
        </div>
        <div class="form-row">
          <div class="form-group">
            <label class="form-label">Type</label>
            <select v-model="newTransaction.type" class="form-input">
              <option value="income">Income</option>
              <option value="expense">Expense</option>
            </select>
          </div>
          <div class="form-group">
            <label class="form-label">Category</label>
            <input 
              v-model="newTransaction.category" 
              type="text" 
              class="form-input" 
              required 
            />
          </div>
        </div>
        <div class="form-group">
          <label class="form-label">Date</label>
          <input 
            v-model="newTransaction.date" 
            type="date" 
            class="form-input" 
            required 
          />
        </div>
        <button type="submit" class="btn btn-success" :disabled="loading">
          {{ loading ? 'Adding...' : 'Add Transaction' }}
        </button>
      </form>
    </div>

    <div class="card">
      <div class="transactions-header">
        <h3>Your Transactions</h3>
        <div class="filters">
          <select v-model="filterType" class="form-input">
            <option value="">All Types</option>
            <option value="income">Income</option>
            <option value="expense">Expense</option>
          </select>
        </div>
      </div>
      
      <div v-if="filteredTransactions.length === 0" class="empty-state">
        <p>No transactions found.</p>
      </div>
      
      <div v-else class="transactions-list">
        <div 
          v-for="transaction in filteredTransactions" 
          :key="transaction._id"
          class="transaction-item"
        >
          <div class="transaction-main">
            <div class="transaction-info">
              <h4>{{ transaction.description }}</h4>
              <span class="category">{{ transaction.category }}</span>
              <span class="date">{{ formatDate(transaction.date) }}</span>
            </div>
            <div class="transaction-amount" :class="transaction.type">
              {{ transaction.type === 'income' ? '+' : '-' }}${{ transaction.amount.toFixed(2) }}
            </div>
          </div>
          <button 
            @click="deleteTransaction(transaction._id)" 
            class="btn btn-danger btn-sm"
            :disabled="loading"
          >
            Delete
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'
import api from '../services/api'

export default {
  name: 'Transactions',
  setup() {
    const transactions = ref([])
    const showAddForm = ref(false)
    const loading = ref(false)
    const filterType = ref('')
    
    const newTransaction = ref({
      description: '',
      amount: 0,
      type: 'expense',
      category: '',
      date: new Date().toISOString().split('T')[0]
    })

    const filteredTransactions = computed(() => {
      if (!filterType.value) return transactions.value
      return transactions.value.filter(t => t.type === filterType.value)
    })

    const loadTransactions = async () => {
      try {
        const response = await api.get('/transactions')
        transactions.value = response.data.transactions.sort((a, b) => 
          new Date(b.date) - new Date(a.date)
        )
      } catch (error) {
        console.error('Error loading transactions:', error)
      }
    }

    const addTransaction = async () => {
      loading.value = true
      try {
        const response = await api.post('/transactions', newTransaction.value)
        transactions.value.unshift(response.data.transaction)
        
        // Reset form
        newTransaction.value = {
          description: '',
          amount: 0,
          type: 'expense',
          category: '',
          date: new Date().toISOString().split('T')[0]
        }
        showAddForm.value = false
      } catch (error) {
        console.error('Error adding transaction:', error)
      }
      loading.value = false
    }

    const deleteTransaction = async (id) => {
      if (!confirm('Are you sure you want to delete this transaction?')) return
      
      loading.value = true
      try {
        await api.delete(`/transactions/${id}`)
        transactions.value = transactions.value.filter(t => t._id !== id)
      } catch (error) {
        console.error('Error deleting transaction:', error)
      }
      loading.value = false
    }

    const formatDate = (dateString) => {
      return new Date(dateString).toLocaleDateString()
    }

    onMounted(loadTransactions)

    return {
      transactions,
      filteredTransactions,
      showAddForm,
      loading,
      filterType,
      newTransaction,
      addTransaction,
      deleteTransaction,
      formatDate
    }
  }
}
</script>

<style scoped>
.page-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
}

.add-form {
  margin-bottom: 2rem;
}

.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
}

.transactions-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}

.filters select {
  width: auto;
  min-width: 150px;
}

.transactions-list {
  space: 1rem 0;
}

.transaction-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem;
  border: 1px solid #eee;
  border-radius: 4px;
  margin-bottom: 0.5rem;
}

.transaction-main {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex: 1;
}

.transaction-info {
  display: flex;
  flex-direction: column;
}

.transaction-info h4 {
  margin: 0 0 0.25rem 0;
  color: #2c3e50;
}

.category {
  font-size: 0.9rem;
  color: #666;
  margin-bottom: 0.25rem;
}

.date {
  font-size: 0.8rem;
  color: #999;
}

.transaction-amount {
  font-size: 1.2rem;
  font-weight: bold;
  margin-right: 1rem;
}

.transaction-amount.income {
  color: #27ae60;
}

.transaction-amount.expense {
  color: #e74c3c;
}

.btn-sm {
  padding: 0.25rem 0.5rem;
  font-size: 0.9rem;
}

.empty-state {
  text-align: center;
  padding: 2rem;
  color: #666;
}
</style>