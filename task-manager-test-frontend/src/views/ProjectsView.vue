<script setup>
import { onMounted, ref } from 'vue'
import api from '@/lib/axios'
const apiUrl = import.meta.env.VITE_API_BASE_URL

const STATUSES = ['PLANNED', 'IN_PROGRESS', 'COMPLETED', 'CANCELLED']
const badge = {
  PLANNED: 'secondary',
  IN_PROGRESS: 'primary',
  COMPLETED: 'success',
  CANCELLED: 'danger',
}

const emptyForm = () => ({
  name: '',
  description: '',
  status: 'PLANNED',
  amount_excl_tax: 0,
  vat_rate: 20,
  start_date: '',
  due_date: '',
  customer_id: '',
})

const projects = ref([])
const customers = ref([])
const form = ref(emptyForm())
const editingId = ref(null)
const error = ref('')
const success = ref('')

const toDateInput = (value) => (value ? value.slice(0, 10) : '')

const load = async () => {
  try {
    const [p, c] = await Promise.all([api.get(`${apiUrl}/projects`), api.get(`${apiUrl}/customers`)])
    projects.value = p.data.projects
    customers.value = c.data.customers
  } catch (e) {
    error.value = e.message
  }
}

const submit = async () => {
  error.value = ''
  // On n'envoie pas start_date vide : l'API utilisera la date du jour
  const payload = { ...form.value }
  if (!payload.start_date) delete payload.start_date
  try {
    const { data } = editingId.value
      ? await api.put(`${apiUrl}/projects/${editingId.value}`, payload)
      : await api.post(`${apiUrl}/projects`, payload)
    success.value = data.message
    reset()
    await load()
  } catch (e) {
    error.value = e.message
  }
}

const edit = (project) => {
  editingId.value = project.id
  form.value = {
    name: project.name,
    description: project.description ?? '',
    status: project.status,
    amount_excl_tax: Number(project.amountExclTax),
    vat_rate: Number(project.vatRate),
    start_date: toDateInput(project.startDate),
    due_date: toDateInput(project.dueDate),
    customer_id: project.customerId,
  }
}

const remove = async (id) => {
  error.value = ''
  try {
    const { data } = await api.delete(`${apiUrl}/projects/${id}`)
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
  <h1 class="h3 mb-4">Projets</h1>

  <div v-if="error" class="alert alert-danger">{{ error }}</div>
  <div v-if="success" class="alert alert-success">{{ success }}</div>

  <div class="row g-4">
    <div class="col-lg-4">
      <div class="card shadow-sm">
        <div class="card-body">
          <h2 class="h5">{{ editingId ? 'Modifier le projet' : 'Nouveau projet' }}</h2>
          <form @submit.prevent="submit">
            <div class="mb-2">
              <label class="form-label">Nom</label>
              <input v-model="form.name" placeholder="Ex. : Refonte du site web" class="form-control" required />
            </div>
            <div class="mb-2">
              <label class="form-label">Description</label>
              <textarea v-model="form.description" placeholder="Décrivez le projet…" class="form-control" rows="2"></textarea>
            </div>
            <div class="mb-2">
              <label class="form-label">Client</label>
              <select v-model="form.customer_id" class="form-select" required>
                <option value="" disabled>Choisir un client</option>
                <option v-for="c in customers" :key="c.id" :value="c.id">{{ c.name }}</option>
              </select>
            </div>
            <div class="mb-2">
              <label class="form-label">Statut</label>
              <select v-model="form.status" class="form-select">
                <option v-for="s in STATUSES" :key="s" :value="s">{{ s }}</option>
              </select>
            </div>
            <div class="row g-2 mb-2">
              <div class="col">
                <label class="form-label">Montant HT</label>
                <input v-model.number="form.amount_excl_tax" placeholder="Ex. : 1500" type="number" step="0.01" min="0" class="form-control" />
              </div>
              <div class="col">
                <label class="form-label">TVA %</label>
                <input v-model.number="form.vat_rate" placeholder="Ex. : 20" type="number" step="0.01" min="0" class="form-control" />
              </div>
            </div>
            <div class="row g-2 mb-3">
              <div class="col">
                <label class="form-label">Début</label>
                <input v-model="form.start_date" type="date" class="form-control" />
              </div>
              <div class="col">
                <label class="form-label">Échéance</label>
                <input v-model="form.due_date" type="date" class="form-control" required />
              </div>
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
            <tr><th>Réf.</th><th>Nom</th><th>Client</th><th>Statut</th><th>Montant HT</th><th>Échéance</th><th></th></tr>
          </thead>
          <tbody>
            <tr v-for="p in projects" :key="p.id">
              <td>{{ p.reference }}</td>
              <td>{{ p.name }}</td>
              <td>{{ p.customer?.name }}</td>
              <td><span :class="`badge bg-${badge[p.status]}`">{{ p.status }}</span></td>
              <td>{{ Number(p.amountExclTax).toFixed(2) }} €</td>
              <td>{{ new Date(p.dueDate).toLocaleDateString('fr-FR') }}</td>
              <td class="text-end text-nowrap">
                <button class="btn btn-sm btn-outline-primary me-1" @click="edit(p)">Modifier</button>
                <button class="btn btn-sm btn-outline-danger" @click="remove(p.id)">Supprimer</button>
              </td>
            </tr>
            <tr v-if="!projects.length">
              <td colspan="7" class="text-center text-muted">Aucun projet</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>
