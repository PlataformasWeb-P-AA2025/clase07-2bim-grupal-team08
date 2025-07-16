<template>
  <div class="list-container">
    <h2>Listado General de Teléfonos</h2>
    <p v-if="loading">Cargando datos...</p>
    <p v-if="error" class="error-message">{{ error }}</p>
    <ul v-else-if="telefonos.length">
      <li
        v-for="telefono in telefonosConNombre"
        :key="telefono.url"
        class="list-item"
      >
        <span>
          <strong>Teléfono:</strong> {{ telefono.telefono }} ({{
            telefono.tipo
          }})
          <small class="student-info">
            - Pertenece a: {{ telefono.nombreEstudiante }}</small
          >
        </span>

        <div class="actions">
          <button @click="abrirModalEdicion(telefono)" class="btn-edit">
            Editar
          </button>
          <button @click="eliminarTelefono(telefono.url)" class="btn-delete">
            Eliminar
          </button>
        </div>
      </li>
    </ul>
    <p v-else>No hay teléfonos registrados en el sistema.</p>
  </div>

  <div v-if="esModalVisible" class="modal-overlay">
    <div class="modal-content">
      <h3>Editar Teléfono</h3>
      <form @submit.prevent="guardarCambios">
        <div class="form-group">
          <label>Número:</label>
          <input type="text" v-model="telefonoParaEditar.telefono" required />
        </div>
        <div class="form-group">
          <label>Tipo:</label>
          <select v-model="telefonoParaEditar.tipo">
            <option>Casa</option>
            <option>Móvil</option>
            <option>Oficina</option>
          </select>
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
  name: "TelefonosList",
  data() {
    return {
      telefonos: [],
      estudiantes: [], // <-- Guardaremos los estudiantes aquí
      loading: true,
      error: null,
      esModalVisible: false,
      telefonoParaEditar: null,
    };
  },
  computed: {
    // Una propiedad computada es perfecta para transformar datos
    telefonosConNombre() {
      // Creamos un mapa para buscar nombres de estudiantes por su URL
      const studentMap = this.estudiantes.reduce((map, student) => {
        map[student.url] = `${student.nombre} ${student.apellido}`;
        return map;
      }, {});

      // Mapeamos la lista de teléfonos para añadir el nombre del estudiante
      return this.telefonos.map((telefono) => ({
        ...telefono,
        // Buscamos el nombre en el mapa, si no lo encuentra, muestra la URL como antes
        nombreEstudiante:
          studentMap[telefono.estudiante] || telefono.estudiante,
      }));
    },
  },
  async created() {
    await this.cargarDatos();
  },
  methods: {
    async cargarDatos() {
      this.loading = true;
      this.error = null;
      try {
        // Hacemos las dos peticiones a la API en paralelo para más eficiencia
        const [responseTelefonos, responseEstudiantes] = await Promise.all([
          api.get("numerosts/"),
          api.get("estudiantes/"),
        ]);

        this.telefonos =
          responseTelefonos.data.results || responseTelefonos.data;
        this.estudiantes =
          responseEstudiantes.data.results || responseEstudiantes.data;
      } catch (err) {
        this.error = "No se pudieron cargar todos los datos.";
        console.error(err);
      } finally {
        this.loading = false;
      }
    },

    async eliminarTelefono(url) {
      if (!confirm("¿Estás seguro de que quieres eliminar este teléfono?")) {
        return;
      }
      try {
        await api.delete(url);
        this.telefonos = this.telefonos.filter((tel) => tel.url !== url);
        alert("Teléfono eliminado con éxito.");
      } catch (err) {
        alert("No se pudo eliminar el teléfono.");
      }
    },

    abrirModalEdicion(telefono) {
      this.telefonoParaEditar = { ...telefono };
      this.esModalVisible = true;
    },

    cerrarModal() {
      this.esModalVisible = false;
      this.telefonoParaEditar = null;
    },

    async guardarCambios() {
      if (!this.telefonoParaEditar) return;
      try {
        const url = this.telefonoParaEditar.url;

        // --- CORRECCIÓN AQUÍ ---
        // 1. Hacemos una copia del objeto a editar.
        const payload = { ...this.telefonoParaEditar };
        // 2. Borramos la propiedad que solo existe en el frontend.
        delete payload.nombreEstudiante;
        // -----------------------

        // 3. Enviamos el objeto limpio (payload) a la API.
        const response = await api.put(url, payload);

        const index = this.telefonos.findIndex((tel) => tel.url === url);
        if (index !== -1) {
          this.telefonos.splice(index, 1, response.data);
        }
        alert("Teléfono actualizado con éxito.");
        this.cerrarModal();
      } catch (error) {
        alert("Hubo un error al actualizar el teléfono.");
      }
    },
  },
};
</script>

<style scoped>
/* Los estilos no necesitan cambios, se mantienen los mismos */
.list-container {
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
.list-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 0;
  border-bottom: 1px solid #eee;
}
.student-info {
  color: #6c757d;
  font-size: 0.8em;
  display: block;
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
.btn-delete {
  background-color: #dc3545;
}
.error-message {
  color: red;
  text-align: center;
}

/* Estilos de Modal (reutilizados) */
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
.modal-actions {
  display: flex;
  justify-content: flex-end;
  margin-top: 20px;
}
.modal-actions button {
  padding: 8px 15px;
  border-radius: 4px;
  border: none;
  cursor: pointer;
  color: white;
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
.form-group input,
.form-group select {
  width: 100%;
  box-sizing: border-box;
  padding: 8px;
}
.form-group label {
  margin-bottom: 0.5rem;
  display: block;
}
</style>
