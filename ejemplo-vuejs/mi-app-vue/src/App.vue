<template>
  <nav>
    <router-link to="/">Home</router-link> |
    <router-link to="/login">Login</router-link> |
    <router-link to="/estudiantes">Estudiantes</router-link>
    <router-link to="/telefonos">Teléfonos</router-link>
    <button v-if="isLoggedIn" @click="logout" class="logout-button">
      Salir
    </button>
  </nav>
  <router-view />
</template>

<script>
export default {
  computed: {
    isLoggedIn() {
      return !!localStorage.getItem("authToken"); // Verifica si hay un token
    },
  },
  methods: {
    logout() {
      localStorage.removeItem("authToken"); // Elimina el token
      this.$router.push("/login"); // Redirige al login
    },
  },
};
</script>
<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

nav {
  padding: 30px;
  background-color: #f8f9fa;
  border-bottom: 1px solid #e7e7e7;
}

nav a {
  font-weight: bold;
  color: #2c3e50;
  text-decoration: none;
  margin: 0 10px;
}

nav a.router-link-exact-active {
  color: #42b983;
}

.logout-button {
  margin-left: 20px;
  padding: 8px 15px;
  background-color: #dc3545;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 0.9em;
}

.logout-button:hover {
  background-color: #c82333;
}
</style>
