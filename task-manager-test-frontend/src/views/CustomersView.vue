<script setup>
import { onMounted, ref } from 'vue'
import api from '@/lib/axios'

const apiUrl = import.meta.env.VITE_API_BASE_URL

const emptyForm = () => ({ name: '', email: '', phone: '', address: '' })

const customers = ref([])
const form = ref(emptyForm())
const editingId = ref(null)
const error = ref('')
const success = ref('')

const load = async () => {
  try {
    const { data } = await api.get(`${apiUrl}/customers`)
    customers.value = data.customers
  } catch (e) {
    error.value = e.message
  }
}

const submit = async () => {
  error.value = ''
  try {
    const { data } = editingId.value
      ? await api.put(`${apiUrl}/customers/${editingId.value}`, form.value)
      : await api.post(`${apiUrl}/customers`, form.value)
    success.value = data.message
    reset()
    await load()
  } catch (e) {
    error.value = e.message
  }
}

const edit = (customer) => {
  editingId.value = customer.id
  form.value = {
    name: customer.name,
    email: customer.email,
    phone: customer.phone ?? '',
    address: customer.address ?? '',
  }
}

const remove = async (id) => {
  error.value = ''
  try {
    const { data } = await api.delete(`${apiUrl}/customers/${id}`)
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
  <h1 class="h3 mb-4">Clients</h1>

  <div v-if="error" class="alert alert-danger">{{ error }}</div>
  <div v-if="success" class="alert alert-success">{{ success }}</div>

  <div class="row g-4">
    <div class="col-lg-4">
      <div class="card shadow-sm">
        <div class="card-body">
          <h2 class="h5">{{ editingId ? 'Modifier le client' : 'Nouveau client' }}</h2>
          <form @submit.prevent="submit">
            <div class="mb-2">
              <label class="form-label">Nom</label>
              <input v-model="form.name" placeholder="Ex. : Acme SARL" class="form-control" required />
            </div>
            <div class="mb-2">
              <label class="form-label">Email</label>
              <input v-model="form.email" placeholder="Ex. : contact@acme.fr" type="email" class="form-control" required />
            </div>
            <div class="mb-2">
              <label class="form-label">Téléphone</label>
              <input v-model="form.phone" placeholder="Ex. : 06 12 34 56 78" class="form-control" />
            </div>
            <div class="mb-3">
              <label class="form-label">Adresse</label>
              <input v-model="form.address" placeholder="Ex. : 12 rue de Paris, 75001 Paris" class="form-control" />
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
            <tr><th>#</th><th>Nom</th><th>Email</th><th>Téléphone</th><th></th></tr>
          </thead>
          <tbody>
            <tr v-for="c in customers" :key="c.id">
              <td>{{ c.id }}</td>
              <td>{{ c.name }}</td>
              <td>{{ c.email }}</td>
              <td>{{ c.phone || '—' }}</td>
              <td class="text-end text-nowrap">
                <button class="btn btn-sm btn-outline-primary me-1" @click="edit(c)">Modifier</button>
                <button class="btn btn-sm btn-outline-danger" @click="remove(c.id)">Supprimer</button>
              </td>
            </tr>
            <tr v-if="!customers.length">
              <td colspan="5" class="text-center text-muted">Aucun client</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>
