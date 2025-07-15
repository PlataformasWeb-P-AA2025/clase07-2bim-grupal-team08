<template>
  <div class="estudiantes-list-container">
    <h2>Listado de Estudiantes</h2>
    <p v-if="loading">Cargando estudiantes...</p>
    <p v-if="error" class="error-message">{{ error }}</p>
    <ul v-else-if="estudiantes.length">
      <li
        v-for="estudiante in estudiantes"
        :key="estudiante.url"
        class="estudiante-item"
      >
        <router-link
          :to="{
            name: 'EstudianteDetail',
            params: { estudianteUrl: estudiante.url },
          }"
        >
          {{ estudiante.nombre }} {{ estudiante.apellido }} (Cédula:
          {{ estudiante.cedula }})
        </router-link>

        <div class="actions">
          <button
            @click.prevent="abrirModalEdicion(estudiante)"
            class="btn-edit"
          >
            Editar
          </button>
          <button
            @click.prevent="eliminarEstudiante(estudiante.url)"
            class="btn-delete"
          >
            Eliminar
          </button>
        </div>
      </li>
    </ul>
    <p v-else>No hay estudiantes registrados.</p>
    <router-link to="/estudiantes/nuevo" class="add-button"
      >Agregar Nuevo Estudiante</router-link
    >
  </div>

  <div v-if="esModalVisible" class="modal-overlay">
    <div class="modal-content">
      <h3>Editar Estudiante</h3>
      <form @submit.prevent="guardarCambios">
        <div class="form-group">
          <label>Nombre:</label>
          <input type="text" v-model="estudianteParaEditar.nombre" required />
        </div>
        <div class="form-group">
          <label>Apellido:</label>
          <input type="text" v-model="estudianteParaEditar.apellido" required />
        </div>
        <div class="form-group">
          <label>Cédula:</label>
          <input type="text" v-model="estudianteParaEditar.cedula" required />
        </div>
        <div class="form-group">
          <label>Correo:</label>
          <input type="email" v-model="estudianteParaEditar.correo" required />
        </div>

        <div class="modal-actions">
          <button type="button" @click="cerrarModal" class="btn-cancel">
            Cancelar
          </button>
          <button type="submit" class="btn-save">Guardar Cambios</button>
        </div>
      </form>
    </div>
  </div>
</template>

<script>
import api from "@/api/axios";

export default {
  name: "EstudiantesList",
  data() {
    return {
      estudiantes: [],
      loading: true,
      error: null,
      esModalVisible: false,
      estudianteParaEditar: null,
    };
  },
  async created() {
    await this.fetchEstudiantes();
  },
  methods: {
    async fetchEstudiantes() {
      try {
        this.loading = true;
        this.error = null;
        const response = await api.get("estudiantes/");
        this.estudiantes = response.data.results || response.data;
      } catch (err) {
        console.error("Error al cargar estudiantes:", err.response || err);
        this.error =
          "No se pudieron cargar los estudiantes. Asegúrate de estar logueado.";
      } finally {
        this.loading = false;
      }
    },

    async eliminarEstudiante(url) {
      if (!confirm("¿Estás seguro de que quieres eliminar este estudiante?")) {
        return;
      }
      try {
        await api.delete(url);
        this.estudiantes = this.estudiantes.filter((est) => est.url !== url);
        alert("Estudiante eliminado con éxito.");
      } catch (err) {
        console.error("Error al eliminar el estudiante:", err.response || err);
        alert("No se pudo eliminar el estudiante.");
      }
    },

    abrirModalEdicion(estudiante) {
      this.estudianteParaEditar = { ...estudiante };
      this.esModalVisible = true;
    },

    cerrarModal() {
      this.esModalVisible = false;
      this.estudianteParaEditar = null;
    },

    async guardarCambios() {
      if (!this.estudianteParaEditar) return;
      try {
        const url = this.estudianteParaEditar.url;
        const response = await api.put(url, this.estudianteParaEditar);
        const index = this.estudiantes.findIndex((est) => est.url === url);
        if (index !== -1) {
          this.estudiantes.splice(index, 1, response.data);
        }
        alert("Estudiante actualizado con éxito.");
        this.cerrarModal();
      } catch (error) {
        console.error("Error al actualizar el estudiante:", error);
        alert("Hubo un error al actualizar.");
      }
    },
  },
};
</script>

<style scoped>
.estudiantes-list-container {
  max-width: 800px;
  margin: 50px auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  background-color: #fff;
}
h2 {
  text-align: center;
  color: #333;
  margin-bottom: 20px;
}
ul {
  list-style: none;
  padding: 0;
}
.estudiante-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 0;
  border-bottom: 1px solid #eee;
}
.estudiante-item:last-child {
  border-bottom: none;
}
.estudiante-item a {
  text-decoration: none;
  color: #007bff;
  font-weight: bold;
}
.estudiante-item a:hover {
  text-decoration: underline;
}
.add-button {
  display: block;
  width: fit-content;
  margin: 20px auto 0;
  padding: 10px 20px;
  background-color: #28a745;
  color: white;
  border-radius: 5px;
  text-decoration: none;
  text-align: center;
  transition: background-color 0.3s ease;
}
.add-button:hover {
  background-color: #218838;
}
.error-message {
  color: red;
  text-align: center;
  margin-top: 10px;
}
.actions button {
  margin-left: 10px;
  padding: 5px 10px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  color: white;
}
.btn-edit {
  background-color: #ffc107;
}
.btn-edit:hover {
  background-color: #e0a800;
}
.btn-delete {
  background-color: #dc3545;
}
.btn-delete:hover {
  background-color: #c82333;
}

/* Estilos para el Modal */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
}
.modal-content {
  background: white;
  padding: 20px 30px;
  border-radius: 8px;
  width: 90%;
  max-width: 500px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
}
.modal-content h3 {
  margin-top: 0;
  margin-bottom: 20px;
}
.modal-actions {
  display: flex;
  justify-content: flex-end;
  margin-top: 20px;
}
.modal-actions button {
  padding: 8px 15px;
}
.btn-cancel {
  background-color: #6c757d;
  margin-right: 10px;
}
.btn-save {
  background-color: #007bff;
}
.form-group {
  margin-bottom: 1rem;
  text-align: left;
}
.form-group input {
  width: 100%;
  box-sizing: border-box; /* Asegura que el padding no afecte el ancho total */
  padding: 8px;
}
.form-group label {
  margin-bottom: 0.5rem;
  display: block;
}
</style>
