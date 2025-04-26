<template>
<div>
    <form @submit.prevent="login">

      <h2>Inicia Sessió</h2>
      <div style="margin-bottom: 1rem;">
        <label>Email</label>
        <input v-model="email" type="email" required />
      </div>
      <div style="margin-bottom: 1rem;">
        <label>Contrasenya</label>
        <input id="password" type="text" v-model="password" required />
      </div>
      <button type="submit">Entrar</button>
      <div v-if="error">{{ error }}</div>
    </form>
</div>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';

const emit = defineEmits(['login-success']);
const email = ref('');
const password = ref('');
const error = ref('');

async function login() {
  error.value = '';
  try {
    const response = await axios.post('http://127.0.0.1:8002/api/login', {
      email: email.value,
      password: password.value,
    });
    emit('login-success', { token: response.data.token, role: response.data.role });
  } catch (e) {
    error.value = 'Credencials incorrectes';
  }
}
</script>
