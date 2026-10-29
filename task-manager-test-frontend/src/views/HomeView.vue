<script setup>
import { onMounted, ref } from 'vue'
import api from '@/lib/axios'

const apiUrl = import.meta.env.VITE_API_BASE_URL
console.log('Test apiUrl', apiUrl)

const stats = ref({ customers: 0, projects: 0, tasks: 0 })
const error = ref('')

onMounted(async () => {
  try {
    const [c, p, t] = await Promise.all([
      api.get(`${apiUrl}/customers`),
      api.get(`${apiUrl}/projects`),
      api.get(`${apiUrl}/tasks`),
    ])
    stats.value = {
      customers: c.data.customers.length,
      projects: p.data.projects.length,
      tasks: t.data.tasks.length,
    }
  } catch (e) {
    error.value = e.message
  }
})

const cards = [
  { key: 'customers', label: 'Clients', to: '/customers', color: 'primary' },
  { key: 'projects', label: 'Projets', to: '/projects', color: 'success' },
  { key: 'tasks', label: 'Tâches', to: '/tasks', color: 'warning' },
]
</script>

<template>
  <h1 class="h3 mb-4">Tableau de bord 3</h1>

  <div v-if="error" class="alert alert-danger">API injoignable : {{ error }}</div>

  <div class="row g-3">
    <div v-for="card in cards" :key="card.key" class="col-md-4">
      <RouterLink :to="card.to" class="text-decoration-none">
        <div :class="`card border-${card.color} shadow-sm`">
          <div class="card-body">
            <h2 class="h6 text-muted">{{ card.label }}</h2>
            <p :class="`display-6 mb-0 text-${card.color}`">{{ stats[card.key] }}</p>
          </div>
        </div>
      </RouterLink>
    </div>
  </div>
</template>
