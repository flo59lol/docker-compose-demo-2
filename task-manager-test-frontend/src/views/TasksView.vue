<script setup>
import { onMounted, ref } from 'vue'
import api from '@/lib/axios'

const apiUrl = import.meta.env.VITE_API_BASE_URL

const STATUSES = ['TODO', 'IN_PROGRESS', 'COMPLETED']
const PRIORITIES = ['LOW', 'MEDIUM', 'HIGH']
const statusBadge = { TODO: 'secondary', IN_PROGRESS: 'primary', COMPLETED: 'success' }
const priorityBadge = { LOW: 'info', MEDIUM: 'warning', HIGH: 'danger' }

const emptyForm = () => ({
  title: '',
  description: '',
  status: 'TODO',
  priority: 'MEDIUM',
  due_date: '',
  project_id: '',
})

const tasks = ref([])
const projects = ref([])
const form = ref(emptyForm())
const editingId = ref(null)
const error = ref('')
const success = ref('')

const projectName = (id) => projects.value.find((p) => p.id === id)?.name ?? '—'

const load = async () => {
  try {
    const [t, p] = await Promise.all([api.get(`${apiUrl}/tasks`), api.get(`${apiUrl}/projects`)])
    tasks.value = t.data.tasks
    projects.value = p.data.projects
  } catch (e) {
    error.value = e.message
  }
}

const submit = async () => {
  error.value = ''
  try {
    const { data } = editingId.value
      ? await api.put(`${apiUrl}/tasks/${editingId.value}`, form.value)
      : await api.post(`${apiUrl}/tasks`, form.value)
    success.value = data.message
    reset()
    await load()
  } catch (e) {
    error.value = e.message
  }
}

const edit = (task) => {
  editingId.value = task.id
  form.value = {
    title: task.title,
    description: task.description ?? '',
    status: task.status,
    priority: task.priority,
    due_date: task.dueDate.slice(0, 10),
    project_id: task.projectId ?? '',
  }
}

const remove = async (id) => {
  error.value = ''
  try {
    const { data } = await api.delete(`${apiUrl}/tasks/${id}`)
    success.value = data.message
    await load()
  } catch (e) {
    error.value = e.message
  }
}

const reset = () => {
  editingId.value = null
  form.value = emptyForm()
}

onMounted(load)
</script>

<template>
  <h1 class="h3 mb-4">Tâches</h1>

  <div v-if="error" class="alert alert-danger">{{ error }}</div>
  <div v-if="success" class="alert alert-success">{{ success }}</div>

  <div class="row g-4">
    <div class="col-lg-4">
      <div class="card shadow-sm">
        <div class="card-body">
          <h2 class="h5">{{ editingId ? 'Modifier la tâche' : 'Nouvelle tâche' }}</h2>
          <form @submit.prevent="submit">
            <div class="mb-2">
              <label class="form-label">Titre</label>
              <input v-model="form.title" placeholder="Ex. : Rédiger le cahier des charges" class="form-control" required minlength="3" />
            </div>
            <div class="mb-2">
              <label class="form-label">Description</label>
              <textarea v-model="form.description" placeholder="Décrivez la tâche…" class="form-control" rows="2"></textarea>
            </div>
            <div class="mb-2">
              <label class="form-label">Projet</label>
              <select v-model="form.project_id" class="form-select">
                <option value="">Aucun projet</option>
                <option v-for="p in projects" :key="p.id" :value="p.id">{{ p.name }}</option>
              </select>
            </div>
            <div class="row g-2 mb-2">
              <div class="col">
                <label class="form-label">Statut</label>
                <select v-model="form.status" class="form-select">
                  <option v-for="s in STATUSES" :key="s" :value="s">{{ s }}</option>
                </select>
              </div>
              <div class="col">
                <label class="form-label">Priorité</label>
                <select v-model="form.priority" class="form-select">
                  <option v-for="p in PRIORITIES" :key="p" :value="p">{{ p }}</option>
                </select>
              </div>
            </div>
            <div class="mb-3">
              <label class="form-label">Échéance</label>
              <input v-model="form.due_date" type="date" class="form-control" required />
            </div>
            <button class="btn btn-primary me-2">{{ editingId ? 'Enregistrer' : 'Créer' }}</button>
            <button v-if="editingId" type="button" class="btn btn-outline-secondary" @click="reset">Annuler</button>
          </form>
        </div>
      </div>
    </div>

    <div class="col-lg-8">
      <div class="table-responsive">
        <table class="table table-hover bg-white shadow-sm align-middle">
          <thead class="table-dark">
            <tr><th>#</th><th>Titre</th><th>Projet</th><th>Statut</th><th>Priorité</th><th>Échéance</th><th></th></tr>
          </thead>
          <tbody>
            <tr v-for="t in tasks" :key="t.id">
              <td>{{ t.id }}</td>
              <td>{{ t.title }}</td>
              <td>{{ projectName(t.projectId) }}</td>
              <td><span :class="`badge bg-${statusBadge[t.status]}`">{{ t.status }}</span></td>
              <td><span :class="`badge bg-${priorityBadge[t.priority]}`">{{ t.priority }}</span></td>
              <td>{{ new Date(t.dueDate).toLocaleDateString('fr-FR') }}</td>
              <td class="text-end text-nowrap">
                <button class="btn btn-sm btn-outline-primary me-1" @click="edit(t)">Modifier</button>
                <button class="btn btn-sm btn-outline-danger" @click="remove(t.id)">Supprimer</button>
              </td>
            </tr>
            <tr v-if="!tasks.length">
              <td colspan="7" class="text-center text-muted">Aucune tâche</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>
