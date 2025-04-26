<template>
  <div>

    <login-form v-if="!isAuthenticated" @login-success="onLoginSuccess" />
    <div v-else>

      <product-table
        :products="products"
        :user-role="userRole"
        :api-token="apiToken"
        @refresh="fetchProducts"
        @edit="onEditProduct"
      />
      <button v-if="userRole === 'admin'" @click="showCreateForm = true">Afegir Producte</button>
      <product-form
        v-if="showCreateForm"
        @close="showCreateForm = false"
        @refresh="fetchProducts"
        :api-token="apiToken"
      />
      <product-form
        v-if="editingProduct"
        :initial-product="editingProduct"
        :api-token="apiToken"
        @close="editingProduct = null"
        @refresh="onEditSuccess"
        edit-mode
      />
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';
import LoginForm from './LoginForm.vue';
import ProductTable from './ProductTable.vue';
import ProductForm from './ProductForm.vue';

const isAuthenticated = ref(false);
const apiToken = ref('');
const userRole = ref('');
const products = ref([]);
const showCreateForm = ref(false);
const editingProduct = ref(null);

function onLoginSuccess(response) {
  // Permite tanto destructuring como objeto response para máxima compatibilidad
  let token = response.token;
  let role = response.role;
  if (response.data) {
    // Axios devuelve .data si es llamada directa
    token = response.data.token;
    role = response.data.role;
  }
  apiToken.value = token;
  userRole.value = role;
  isAuthenticated.value = true;
  fetchProducts();
}

async function fetchProducts() {
  if (!apiToken.value) return;
  try {
    const response = await axios.get('http://127.0.0.1:8002/api/products', {
      headers: {
        Authorization: `Bearer ${apiToken.value}`
      }
    });
    products.value = response.data;
  } catch (e) {
    products.value = [];
  }
}
function onEditProduct(product) {
  editingProduct.value = { ...product };
}

function onEditSuccess() {
  editingProduct.value = null;
  fetchProducts();
}
</script>
