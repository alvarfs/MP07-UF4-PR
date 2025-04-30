<template>
  <div>
    <table>
      <thead>
        <tr>
          <th @click="sort('name')">Nom <span v-if="sortKey==='name'">{{ sortDir==='asc'?'▲':'▼' }}</span></th>
          <th @click="sort('price')">Preu <span v-if="sortKey==='price'">{{ sortDir==='asc'?'▲':'▼' }}</span></th>
          <th v-if="userRole==='admin'">Accions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="product in sortedProducts" :key="product.id">
          <td>{{ product.name }}</td>
          <td>{{ product.price }} €</td>
          <td v-if="userRole==='admin'">
            <button @click="emit('edit', product)">Editar</button>
            <button @click="deleteProduct(product.id)">Eliminar</button>
          </td>
        </tr>
      </tbody>
    </table>
    <div v-if="error" style="color: #dc2626; margin-top: 1rem;">
      {{ error }}
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue';
import axios from 'axios';

const error = ref('');

const props = defineProps({
  products: Array,
  userRole: String,
  apiToken: String,
});

const sortKey = ref('name');
const sortDir = ref('asc');

function sort(key) {
  if (sortKey.value === key) {
    sortDir.value = sortDir.value === 'asc' ? 'desc' : 'asc';
  } else {
    sortKey.value = key;
    sortDir.value = 'asc';
  }
}

const sortedProducts = computed(() => {
  return [...props.products].sort((a, b) => {
    let cmp = 0;
    if (sortKey.value === 'name') {
      cmp = a.name.localeCompare(b.name);
    } else if (sortKey.value === 'price') {
      cmp = a.price - b.price;
    }
    return sortDir.value === 'asc' ? cmp : -cmp;
  });
});

const emit = defineEmits(['refresh', 'edit']);

async function deleteProduct(id) {
  error.value = '';
  if (!confirm('Segur que vols eliminar aquest producte?')) return;
  // S'hauria de passar el token via prop o context
  // Aquí només emetem l'event perquè ProductApp ho gestioni
  // Això permet controlar el token i la recàrrega des del component pare
  // També podríem passar apiToken via prop si cal
  // $emit('delete', id)
  // Però per simplicitat, cridem l'event refresh per recarregar després de l'eliminació
  try {
    await axios.delete(`http://127.0.0.1:8000/api/products/${id}`, {
      headers: {
        Authorization: `Bearer ${props.apiToken}`
      }
    });
    // Notifiquem al pare que refresqui
    emit('refresh');
  } catch (e) {
    // Muestra el mensaje real de la API si existe
    if (e.response && e.response.data && e.response.data.message) {
      error.value = e.response.data.message;
    } else if (e.message) {
      error.value = 'Error eliminant producte: ' + e.message;
    } else if (e.response && e.response.status) {
      error.value = 'Error eliminant producte. Codi: ' + e.response.status;
    } else {
      error.value = 'Error eliminant producte (desconegut)';
    }
    // También lo mostramos en consola para depuración
    console.error('Error eliminant producte:', e);
  }
}

// Permet que el pare pugui refrescar la taula
watch(() => props.products, () => {}, { deep: true });
</script>
