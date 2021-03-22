<template>
  <div class="budgets">
    <div class="page-header">
      <h1>Budgets</h1>
      <button @click="showAddForm = !showAddForm" class="btn btn-primary">
        {{ showAddForm ? 'Cancel' : 'Create Budget' }}
      </button>
    </div>

    <div v-if="showAddForm" class="card add-form">
      <h3>Create New Budget</h3>
      <form @submit.prevent="addBudget">
        <div class="form-row">
          <div class="form-group">
            <label class="form-label">Category</label>
            <input 
              v-model="newBudget.category" 
              type="text" 
              class="form-input" 
              required 
            />
          </div>
          <div class="form-group">
            <label class="form-label">Amount</label>
            <input 
              v-model.number="newBudget.amount" 
              type="number" 
              step="0.01"
              class="form-input" 
              required 
            />
          </div>
        </div>
        <div class="form-row">
          <div class="form-group">
            <label class="form-label">Period</label>
            <select v-model="newBudget.period" class="form-input">
              <option value="monthly">Monthly</option>
              <option value="yearly">Yearly</option>
            </select>
          </div>
          <div class="form-group">
            <label class="form-label">Alert Threshold (%)</label>
            <input 
              v-model.number="newBudget.alertThreshold" 
              type="number" 
              min="1"
              max="100"
              class="form-input" 
              required 
            />
          </div>
        </div>
        <button type="submit" class="btn btn-success" :disabled="loading">
          {{ loading ? 'Creating...' : 'Create Budget' }}
        </button>
      </form>
    </div>

    <div v-if="budgets.length === 0" class="empty-state card">
      <p>No budgets created yet.</p>
      <p>Create your first budget to start tracking your spending!</p>
    </div>

    <div v-else class="budgets-grid">
      <div 
        v-for="budget in budgets" 
        :key="budget._id"
        class="budget-card card"
      >
        <div class="budget-header">
          <h3>{{ budget.category }}</h3>
          <span class="budget-period">{{ budget.period }}</span>
        </div>
        
        <div class="budget-amount">
          <span class="current">${{ budget.spent.toFixed(2) }}</span>
          <span class="separator"> / </span>
          <span class="total">${{ budget.amount.toFixed(2) }}</span>
        </div>
        
        <div class="progress-bar">
          <div 
            class="progress-fill" 
            :class="{ 'over-budget': budget.spent > budget.amount }"
            :style="{ width: Math.min(progressPercentage(budget), 100) + '%' }"
          ></div>
        </div>
        
        <div class="budget-status">
          <span :class="getStatusClass(budget)">
            {{ getBudgetStatus(budget) }}
          </span>
          <span class="percentage">
            {{ progressPercentage(budget).toFixed(1) }}%
          </span>
        </div>
        
        <div class="budget-actions">
          <button 
            @click="toggleBudget(budget._id, budget.isActive)"
            class="btn btn-sm"
            :class="budget.isActive ? 'btn-danger' : 'btn-success'"
          >
            {{ budget.isActive ? 'Deactivate' : 'Activate' }}
          </button>
          <button 
            @click="deleteBudget(budget._id)" 
            class="btn btn-danger btn-sm"
          >
            Delete
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue'
import api from '../services/api'

export default {
  name: 'Budgets',
  setup() {
    const budgets = ref([])
    const showAddForm = ref(false)
    const loading = ref(false)
    
    const newBudget = ref({
      category: '',
      amount: 0,
      period: 'monthly',
      alertThreshold: 80
    })

    const loadBudgets = async () => {
      try {
        const response = await api.get('/budgets')
        budgets.value = response.data.budgets
      } catch (error) {
        console.error('Error loading budgets:', error)
      }
    }

    const addBudget = async () => {
      loading.value = true
      try {
        const response = await api.post('/budgets', newBudget.value)
        budgets.value.push(response.data.budget)
        
        // Reset form
        newBudget.value = {
          category: '',
          amount: 0,
          period: 'monthly',
          alertThreshold: 80
        }
        showAddForm.value = false
      } catch (error) {
        console.error('Error adding budget:', error)
      }
      loading.value = false
    }

    const deleteBudget = async (id) => {
      if (!confirm('Are you sure you want to delete this budget?')) return
      
      try {
        await api.delete(`/budgets/${id}`)
        budgets.value = budgets.value.filter(b => b._id !== id)
      } catch (error) {
        console.error('Error deleting budget:', error)
      }
    }

    const toggleBudget = async (id, currentStatus) => {
      try {
        const response = await api.put(`/budgets/${id}`, {
          isActive: !currentStatus
        })
        const index = budgets.value.findIndex(b => b._id === id)
        if (index !== -1) {
          budgets.value[index] = response.data.budget
        }
      } catch (error) {
        console.error('Error updating budget:', error)
      }
    }

    const progressPercentage = (budget) => {
      return budget.amount > 0 ? (budget.spent / budget.amount) * 100 : 0
    }

    const getBudgetStatus = (budget) => {
      const percentage = progressPercentage(budget)
      if (!budget.isActive) return 'Inactive'
      if (percentage >= 100) return 'Over Budget'
      if (percentage >= budget.alertThreshold) return 'Alert'
      return 'On Track'
    }

    const getStatusClass = (budget) => {
      const status = getBudgetStatus(budget)
      if (status === 'Over Budget') return 'status-over'
      if (status === 'Alert') return 'status-warning'
      if (status === 'Inactive') return 'status-inactive'
      return 'status-good'
    }

    onMounted(loadBudgets)

    return {
      budgets,
      showAddForm,
      loading,
      newBudget,
      addBudget,
      deleteBudget,
      toggleBudget,
      progressPercentage,
      getBudgetStatus,
      getStatusClass
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

.budgets-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 1rem;
}

.budget-card {
  position: relative;
}

.budget-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}

.budget-header h3 {
  margin: 0;
  color: #2c3e50;
}

.budget-period {
  font-size: 0.9rem;
  color: #666;
  background: #f8f9fa;
  padding: 0.25rem 0.5rem;
  border-radius: 4px;
}

.budget-amount {
  text-align: center;
  margin-bottom: 1rem;
}

.current {
  font-size: 1.5rem;
  font-weight: bold;
  color: #2c3e50;
}

.separator {
  color: #666;
  margin: 0 0.5rem;
}

.total {
  font-size: 1.2rem;
  color: #666;
}

.progress-bar {
  height: 8px;
  background: #ecf0f1;
  border-radius: 4px;
  overflow: hidden;
  margin-bottom: 1rem;
}

.progress-fill {
  height: 100%;
  background: #3498db;
  transition: width 0.3s ease;
}

.progress-fill.over-budget {
  background: #e74c3c;
}

.budget-status {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}

.status-good { color: #27ae60; }
.status-warning { color: #f39c12; }
.status-over { color: #e74c3c; }
.status-inactive { color: #95a5a6; }

.percentage {
  font-weight: bold;
}

.budget-actions {
  display: flex;
  gap: 0.5rem;
}

.budget-actions .btn {
  flex: 1;
}

.btn-sm {
  padding: 0.25rem 0.5rem;
  font-size: 0.9rem;
}

.empty-state {
  text-align: center;
  padding: 3rem 2rem;
  color: #666;
}

.empty-state p {
  margin-bottom: 1rem;
}
</style>