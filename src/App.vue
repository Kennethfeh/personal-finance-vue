<template>
  <div id="app">
    <nav class="navbar">
      <div class="nav-brand">
        <h1>💰 Personal Finance</h1>
      </div>
      <ul class="nav-menu" v-if="authStore.isAuthenticated">
        <li><router-link to="/dashboard">Dashboard</router-link></li>
        <li><router-link to="/transactions">Transactions</router-link></li>
        <li><router-link to="/budgets">Budgets</router-link></li>
        <li><button @click="logout">Logout</button></li>
      </ul>
    </nav>
    
    <main class="main-content">
      <router-view />
    </main>
  </div>
</template>

<script>
import { useAuthStore } from './stores/auth'

export default {
  name: 'App',
  setup() {
    const authStore = useAuthStore()
    
    const logout = () => {
      authStore.logout()
    }

    return {
      authStore,
      logout
    }
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background-color: #f5f5f5;
}

.navbar {
  background: #2c3e50;
  color: white;
  padding: 1rem 2rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.nav-brand h1 {
  font-size: 1.5rem;
}

.nav-menu {
  display: flex;
  list-style: none;
  gap: 2rem;
  align-items: center;
}

.nav-menu a {
  color: white;
  text-decoration: none;
  padding: 0.5rem 1rem;
  border-radius: 4px;
  transition: background-color 0.3s;
}

.nav-menu a:hover {
  background-color: #34495e;
}

.nav-menu a.router-link-active {
  background-color: #3498db;
}

.nav-menu button {
  background: #e74c3c;
  color: white;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.nav-menu button:hover {
  background-color: #c0392b;
}

.main-content {
  padding: 2rem;
  max-width: 1200px;
  margin: 0 auto;
}
</style>