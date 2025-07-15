<template>
  <div class="form-container">
    <h2>Agregar Nuevo Estudiante</h2>
    <form @submit.prevent="agregarEstudiante">
      <div class="form-group">
        <label for="nombre">Nombre:</label>
        <input type="text" id="nombre" v-model="estudiante.nombre" required />
      </div>
      <div class="form-group">
        <label for="apellido">Apellido:</label>
        <input
          type="text"
          id="apellido"
          v-model="estudiante.apellido"
          required
        />
      </div>
      <div class="form-group">
        <label for="cedula">Cédula:</label>
        <input type="text" id="cedula" v-model="estudiante.cedula" required />
      </div>
      <div class="form-group">
        <label for="correo">Correo Electrónico:</label>
        <input type="email" id="correo" v-model="estudiante.correo" required />
      </div>
      <button type="submit">Guardar Estudiante</button>
      <p v-if="error" class="error-message">{{ error }}</p>
    </form>
  </div>
</template>

<script>
import api from "@/api/axios"; // Importamos nuestra instancia de Axios
export default {
  name: "EstudianteNuevo",
  data() {
    return {
      estudiante: {
        nombre: "",
        apellido: "",
        cedula: "",
        correo: "",
      },
      error: null,
    };
  },
  methods: {
    async agregarEstudiante() {
      try {
        this.error = null;
        // Hacemos una petición POST al endpoint de estudiantes
        await api.post("estudiantes/", this.estudiante);

        alert("¡Estudiante agregado con éxito!");

        // Redirigimos al usuario de vuelta a la lista de estudiantes
        this.$router.push("/estudiantes");
      } catch (err) {
        console.error("Error al agregar el estudiante:", err.response || err);
        this.error = "No se pudo agregar el estudiante. Verifique los datos.";
      }
    },
  },
};
</script>

<style scoped>
.form-container {
  max-width: 500px;
  margin: 50px auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}
.form-group {
  margin-bottom: 15px;
}
label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}
input {
  width: calc(100% - 22px);
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
}
button {
  width: 100%;
  padding: 12px;
  background-color: #28a745;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1em;
}
button:hover {
  background-color: #218838;
}
.error-message {
  color: red;
  text-align: center;
  margin-top: 10px;
}
</style>
