<template>
  <div class="estudiante-detail-container">
    <h2>Detalle del Estudiante</h2>
    <p v-if="loading">Cargando detalles...</p>
    <p v-if="error" class="error-message">{{ error }}</p>

    <div v-else-if="estudiante">
      <h3>{{ estudiante.nombre }} {{ estudiante.apellido }}</h3>
      <p><strong>Cédula:</strong> {{ estudiante.cedula }}</p>
      <p><strong>Correo:</strong> {{ estudiante.correo }}</p>

      <h4>Números Telefónicos:</h4>
      <ul v-if="numerosTelefonicos.length" class="phone-list">
        <li
          v-for="numero in numerosTelefonicos"
          :key="numero.url"
          class="phone-item"
        >
          <span>{{ numero.telefono }} ({{ numero.tipo }})</span>
          <div class="phone-actions">
            <button
              @click="abrirModalEditar(numero)"
              class="btn-small btn-edit"
            >
              Editar
            </button>
            <button
              @click="eliminarTelefono(numero)"
              class="btn-small btn-delete"
            >
              Eliminar
            </button>
          </div>
        </li>
      </ul>
      <p v-else>No tiene números telefónicos registrados.</p>
      <button @click="abrirModalAgregar" class="add-phone-button">
        Agregar Nuevo Teléfono
      </button>

      <router-link :to="{ name: 'EstudiantesList' }" class="back-button"
        >Volver al Listado</router-link
      >
    </div>
    <p v-else>Estudiante no encontrado.</p>
  </div>

  <div v-if="esModalEditarVisible" class="modal-overlay">
    <div class="modal-content">
      <h3>Editar Teléfono</h3>
      <form @submit.prevent="guardarCambiosTelefono">
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
          <button type="button" @click="cerrarModalEditar" class="btn-cancel">
            Cancelar
          </button>
          <button type="submit" class="btn-save">Guardar Cambios</button>
        </div>
      </form>
    </div>
  </div>

  <div v-if="esModalAgregarVisible" class="modal-overlay">
    <div class="modal-content">
      <h3>Agregar Nuevo Teléfono</h3>
      <form @submit.prevent="agregarNuevoTelefono">
        <div class="form-group">
          <label>Número:</label>
          <input type="text" v-model="nuevoTelefono.telefono" required />
        </div>
        <div class="form-group">
          <label>Tipo:</label>
          <select v-model="nuevoTelefono.tipo">
            <option>Casa</option>
            <option>Móvil</option>
            <option>Oficina</option>
          </select>
        </div>
        <div class="modal-actions">
          <button type="button" @click="cerrarModalAgregar" class="btn-cancel">
            Cancelar
          </button>
          <button type="submit" class="btn-save">Agregar</button>
        </div>
      </form>
    </div>
  </div>
</template>

<script>
import api from "@/api/axios";

export default {
  name: "EstudianteDetail",
  props: ["estudianteUrl"],
  data() {
    return {
      estudiante: null,
      numerosTelefonicos: [],
      loading: true,
      error: null,
      // State para los modales de teléfono
      esModalEditarVisible: false,
      esModalAgregarVisible: false,
      telefonoParaEditar: null,
      nuevoTelefono: { telefono: "", tipo: "Casa" },
    };
  },
  async created() {
    const decodedUrl = decodeURIComponent(this.estudianteUrl);
    await this.fetchEstudianteDetail(decodedUrl);
    await this.fetchNumerosTelefonicos(decodedUrl);
  },
  methods: {
    async fetchEstudianteDetail(url) {
      try {
        this.loading = true;
        this.error = null;
        const response = await api.get(url);
        this.estudiante = response.data;
      } catch (err) {
        this.error = "No se pudo cargar el detalle del estudiante.";
      } finally {
        this.loading = false;
      }
    },
    async fetchNumerosTelefonicos(estudianteApiUrl) {
      try {
        const response = await api.get("numerosts/");
        this.numerosTelefonicos = response.data.results.filter(
          (numero) => numero.estudiante === estudianteApiUrl
        );
      } catch (err) {
        console.error("Error al cargar números telefónicos:", err);
      }
    },

    // --- MÉTODOS PARA CRUD DE TELÉFONOS ---

    async eliminarTelefono(numero) {
      if (
        !confirm(`¿Seguro que quieres eliminar el teléfono ${numero.telefono}?`)
      )
        return;
      try {
        await api.delete(numero.url);
        this.numerosTelefonicos = this.numerosTelefonicos.filter(
          (n) => n.url !== numero.url
        );
        alert("Teléfono eliminado.");
      } catch (error) {
        alert("Error al eliminar el teléfono.");
      }
    },

    abrirModalEditar(telefono) {
      this.telefonoParaEditar = { ...telefono };
      this.esModalEditarVisible = true;
    },
    cerrarModalEditar() {
      this.esModalEditarVisible = false;
      this.telefonoParaEditar = null;
    },
    async guardarCambiosTelefono() {
      try {
        const response = await api.put(
          this.telefonoParaEditar.url,
          this.telefonoParaEditar
        );
        const index = this.numerosTelefonicos.findIndex(
          (n) => n.url === this.telefonoParaEditar.url
        );
        if (index !== -1) {
          this.numerosTelefonicos.splice(index, 1, response.data);
        }
        this.cerrarModalEditar();
        alert("Teléfono actualizado.");
      } catch (error) {
        alert("Error al actualizar el teléfono.");
      }
    },

    abrirModalAgregar() {
      this.nuevoTelefono = { telefono: "", tipo: "Casa" };
      this.esModalAgregarVisible = true;
    },
    cerrarModalAgregar() {
      this.esModalAgregarVisible = false;
    },
    async agregarNuevoTelefono() {
      try {
        const payload = {
          ...this.nuevoTelefono,
          estudiante: this.estudiante.url, // Asociamos el teléfono al estudiante actual
        };
        const response = await api.post("numerosts/", payload);
        this.numerosTelefonicos.push(response.data);
        this.cerrarModalAgregar();
        alert("Teléfono agregado con éxito.");
      } catch (error) {
        alert("Error al agregar el teléfono.");
      }
    },
  },
};
</script>

<style scoped>
/* --- Estilos existentes --- */
.estudiante-detail-container {
  max-width: 600px;
  margin: 50px auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  background-color: #fff;
  text-align: left;
}
h2,
h3,
h4 {
  text-align: center;
  color: #333;
  margin-bottom: 15px;
}
p {
  margin-bottom: 10px;
}
.back-button {
  display: block;
  width: fit-content;
  margin: 20px auto 0;
  padding: 10px 20px;
  background-color: #007bff;
  color: white;
  border-radius: 5px;
  text-decoration: none;
  text-align: center;
}
.error-message {
  color: red;
  text-align: center;
}

/* --- Nuevos estilos para la lista de teléfonos --- */
.phone-list {
  list-style: none;
  padding: 0;
}
.phone-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px;
  border-bottom: 1px solid #eee;
}
.phone-actions .btn-small {
  padding: 4px 8px;
  font-size: 0.8em;
  margin-left: 5px;
  border: none;
  border-radius: 4px;
  color: white;
  cursor: pointer;
}
.btn-edit {
  background-color: #ffc107;
}
.btn-delete {
  background-color: #dc3545;
}
.add-phone-button {
  display: inline-block;
  margin-top: 15px;
  padding: 8px 12px;
  background-color: #17a2b8;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

/* --- Estilos para los modales (reutilizados) --- */
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
  max-width: 400px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
}
.form-group {
  margin-bottom: 1rem;
  text-align: left;
}
.form-group label {
  margin-bottom: 0.5rem;
  display: block;
}
.form-group input,
.form-group select {
  width: 100%;
  box-sizing: border-box;
  padding: 8px;
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
</style>
