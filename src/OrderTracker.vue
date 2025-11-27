<template>
  <div class="tracker-container">
    <h1>Rastrea tu Pedido</h1>

    <div class="search-card">
      <form @submit.prevent="fetchOrder">
        <div class="form-group">
          <label>Número de Factura (Invoice #)</label>
          <input 
            v-model="form.invoice_number" 
            type="text" 
            placeholder="Ej: INV-2023-001" 
            required 
          />
        </div>

        <div class="form-group">
          <label>Número de Cliente</label>
          <input 
            v-model="form.customer_number" 
            type="text" 
            placeholder="Ej: CUST-9988" 
            required 
          />
        </div>

        <button type="submit" :disabled="loading">
          {{ loading ? 'Buscando...' : 'Rastrear Orden' }}
        </button>
      </form>
      
      <p v-if="error" class="error-msg">{{ error }}</p>
    </div>

    <!-- Resultados -->
    <div v-if="order" class="result-card">
      <div class="header-status">
        <h2>Orden #{{ order.invoice_number }}</h2>
        <!-- Manejo de clases dinámicas para colores de estado -->
        <span :class="['status-badge', getStatusClass(order.status)]">
          {{ order.status }}
        </span>
      </div>

      <div class="details-grid">
        <div class="detail-item">
          <strong>Fecha:</strong>
          <span>{{ formatDate(order.order_date) }}</span>
        </div>
        <div class="detail-item">
          <strong>Total:</strong>
          <!-- Verificamos si existe total_amount -->
          <span>${{ order.total_amount || '0.00' }}</span>
        </div>
        <div class="detail-item">
          <strong>Dirección de Entrega:</strong>
          <span>{{ order.delivery_address }}</span>
        </div>
        <div class="detail-item">
            <strong>Cliente:</strong>
            <span>{{ order.customer.name }} ({{ order.customer.customer_number }})</span>
        </div>
      </div>

      <h3>Productos</h3>
      <table class="products-table">
        <thead>
          <tr>
            <th>Producto</th>
            <th>Cant.</th>
            <th>Precio Unit.</th>
            <th>Subtotal</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="product in order.products" :key="product.id">
            <td>{{ product.title }}</td>
            <!-- Usamos pivot para acceder a los datos de la tabla intermedia -->
            <td>{{ product.pivot?.quantity || product.quantity }}</td>
            <td>${{ product.pivot?.unit_price || product.unit_price }}</td>
            <td>${{ ((product.pivot?.quantity || 0) * (product.pivot?.unit_price || 0)).toFixed(2) }}</td>
          </tr>
        </tbody>
      </table>

      <!-- Sección de Fotos -->
      <div v-if="order.photos && order.photos.length > 0" class="photos-section">
        <h3>Evidencia de Entrega / Carga</h3>
        <div class="photo-grid">
          <div v-for="photo in order.photos" :key="photo.id" class="photo-card">
            <!-- Asumimos que la URL viene completa o es relativa a storage -->
            <img :src="photo.photo_url || photo.photo_path" alt="Evidencia" />
            <span class="photo-type">{{ photo.photo_type }}</span>
            <small>{{ formatDate(photo.uploaded_at) }}</small>
          </div>
        </div>
      </div>
      <div v-else class="no-photos">
          <p><em>No hay fotos disponibles para esta orden aún.</em></p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';

// Estado Reactivo
const form = ref({
  invoice_number: '',
  customer_number: ''
});

const order = ref(null);
const loading = ref(false);
const error = ref(null);

// Métodos
const fetchOrder = async () => {
  loading.value = true;
  error.value = null;
  order.value = null;

  try {
    // PASO 1: Usamos la ruta existente de búsqueda (GET /statuses)
    // Filtramos por invoice_number
    const searchResponse = await axios.get('http://127.0.0.1:8000/api/orders/statuses', {
      params: {
        invoice_number: form.value.invoice_number,
        per_page: 5 // Traemos pocos, debería ser único
      }
    });

    const results = searchResponse.data.data; // Laravel paginación devuelve data dentro de data

    if (!results || results.length === 0) {
      throw new Error('NOT_FOUND');
    }

    // PASO 2: Verificación manual en el Frontend (Lógica de negocio)
    // Buscamos si alguno de los resultados coincide con el Customer Number ingresado
    const match = results.find(o => 
        o.customer && 
        o.customer.customer_number.toString().toLowerCase() === form.value.customer_number.toString().toLowerCase()
    );

    if (!match) {
        throw new Error('NOT_FOUND');
    }

    // PASO 3: Si coincide, obtenemos el detalle completo usando el ID (GET /orders/{id})
    // Hacemos esto para asegurar que traemos productos y fotos que quizás no vienen en el listado
    const detailResponse = await axios.get(`http://127.0.0.1:8000/api/orders/${match.order_id || match.id}`);
    
    if(detailResponse.data.success) {
        order.value = detailResponse.data.data;
    } else {
        throw new Error('ERROR_DETAILS');
    }

  } catch (err) {
    console.error(err);
    if (err.message === 'NOT_FOUND') {
      error.value = "No encontramos una orden que coincida con ese Número de Factura y Cliente.";
    } else {
      error.value = "Ocurrió un error al buscar la información. Intente nuevamente.";
    }
  } finally {
    loading.value = false;
  }
};

const formatDate = (dateString) => {
  if (!dateString) return '';
  const options = { year: 'numeric', month: 'long', day: 'numeric', hour: '2-digit', minute:'2-digit' };
  return new Date(dateString).toLocaleDateString('es-ES', options);
};

const getStatusClass = (status) => {
    if (!status) return '';
    return status.toLowerCase().replace(/\s+/g, '-');
};
</script>

<style scoped>
.tracker-container {
  max-width: 800px;
  margin: 2rem auto;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  padding: 0 1rem;
}

.search-card, .result-card {
  background: white;
  padding: 2rem;
  border-radius: 12px;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
  margin-bottom: 2rem;
}

.form-group {
  margin-bottom: 1.5rem;
}

label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 600;
  color: #374151;
}

input {
  width: 100%;
  padding: 0.75rem;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  box-sizing: border-box; 
  font-size: 1rem;
  transition: border-color 0.2s;
}

input:focus {
    outline: none;
    border-color: #3b82f6;
    box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
}

button {
  width: 100%;
  padding: 0.75rem;
  background-color: #2563eb;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 1rem;
  font-weight: 600;
  transition: background-color 0.2s;
}

button:hover:not(:disabled) {
  background-color: #1d4ed8;
}

button:disabled {
  background-color: #93c5fd;
  cursor: not-allowed;
}

.error-msg {
  color: #dc2626;
  background-color: #fee2e2;
  padding: 1rem;
  border-radius: 6px;
  margin-top: 1rem;
  text-align: center;
  border: 1px solid #fecaca;
}

.header-status {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 2px solid #f3f4f6;
  padding-bottom: 1rem;
  margin-bottom: 1.5rem;
  flex-wrap: wrap;
  gap: 10px;
}

h2 { margin: 0; color: #111827; }

.status-badge {
  padding: 0.5rem 1rem;
  border-radius: 9999px;
  font-weight: 600;
  color: white;
  font-size: 0.875rem;
  background-color: #6b7280;
  text-transform: capitalize;
}

/* Colores de estado basados en tus Enums */
.ordered { background-color: #3b82f6; } /* Azul */
.in-process { background-color: #f59e0b; } /* Naranja */
.in-route { background-color: #8b5cf6; } /* Violeta */
.delivered { background-color: #10b981; } /* Verde */

.details-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1.5rem;
  margin-bottom: 2rem;
}

.detail-item strong {
    display: block;
    color: #6b7280;
    font-size: 0.875rem;
    margin-bottom: 0.25rem;
}

.detail-item span {
    font-size: 1.1rem;
    color: #111827;
    font-weight: 500;
}

.products-table {
  width: 100%;
  border-collapse: separate;
  border-spacing: 0;
  margin-top: 1rem;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  overflow: hidden;
}

.products-table th {
    background-color: #f9fafb;
    font-weight: 600;
    text-transform: uppercase;
    font-size: 0.75rem;
    letter-spacing: 0.05em;
    color: #6b7280;
}

.products-table th, .products-table td {
  text-align: left;
  padding: 1rem;
  border-bottom: 1px solid #e5e7eb;
}

.products-table tr:last-child td {
    border-bottom: none;
}

.photo-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 1.5rem;
  margin-top: 1rem;
}

.photo-card {
    border: 1px solid #e5e7eb;
    border-radius: 8px;
    padding: 0.5rem;
    background: #f9fafb;
    display: flex;
    flex-direction: column;
}

.photo-card img {
  width: 100%;
  height: 150px;
  object-fit: cover;
  border-radius: 4px;
  margin-bottom: 0.5rem;
  cursor: pointer;
}

.photo-type {
    background-color: #e5e7eb;
    color: #374151;
    padding: 2px 8px;
    border-radius: 4px;
    font-size: 0.75rem;
    font-weight: bold;
    text-transform: uppercase;
    align-self: flex-start;
    margin-bottom: 4px;
}
</style>