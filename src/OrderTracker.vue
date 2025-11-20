<template>
  <div class="tracker-container">
    <h1>Track Your Order</h1>

    <div class="search-card">
      <form @submit.prevent="fetchOrder">
        <div class="form-group">
          <label>Order Number (Invoice #)</label>
          <input 
            v-model="form.invoice_number" 
            type="text" 
            placeholder="e.g., INV-2023-001" 
            required 
          />
        </div>

        <div class="form-group">
          <label>Customer Number</label>
          <input 
            v-model="form.customer_number" 
            type="text" 
            placeholder="e.g., CUST-9988" 
            required 
          />
        </div>

        <button type="submit" :disabled="loading">
          {{ loading ? 'Searching...' : 'Track Order' }}
        </button>
      </form>
      
      <p v-if="error" class="error-msg">{{ error }}</p>
    </div>

    <div v-if="order" class="result-card">
      <div class="header-status">
        <h2>Order #{{ order.invoice_number }}</h2>
        <span :class="['status-badge', order.status.toLowerCase().replace(' ', '-')]">
          {{ order.status }}
        </span>
      </div>

      <div class="details-grid">
        <div class="detail-item">
          <strong>Order Date:</strong>
          <span>{{ formatDate(order.order_date) }}</span>
        </div>
        <div class="detail-item">
          <strong>Total Amount:</strong>
          <span>${{ order.total_amount }}</span>
        </div>
        <div class="detail-item">
          <strong>Delivery Address:</strong>
          <span>{{ order.delivery_address }}</span>
        </div>
      </div>

      <h3>Items Ordered</h3>
      <table class="products-table">
        <thead>
          <tr>
            <th>Product</th>
            <th>Qty</th>
            <th>Unit Price</th>
            <th>Subtotal</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="product in order.products" :key="product.id">
            <td>{{ product.title }}</td>
            <td>{{ product.pivot.quantity }}</td>
            <td>${{ product.pivot.unit_price }}</td>
            <td>${{ (product.pivot.quantity * product.pivot.unit_price).toFixed(2) }}</td>
          </tr>
        </tbody>
      </table>

      <div v-if="order.photos && order.photos.length > 0" class="photos-section">
        <h3>Proof of Delivery / Loading</h3>
        <div class="photo-grid">
          <div v-for="photo in order.photos" :key="photo.id" class="photo-card">
            <img :src="photo.photo_path" alt="Order Photo" />
            <span>{{ photo.photo_type }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';

// Reactive State
const form = ref({
  invoice_number: '',
  customer_number: ''
});

const order = ref(null);
const loading = ref(false);
const error = ref(null);

// Methods
const fetchOrder = async () => {
  loading.value = true;
  error.value = null;
  order.value = null;

  try {
    // Adjust localhost URL to your actual Laravel API URL
    const response = await axios.post('http://127.0.0.1:8000/api/orders/track', {
    invoice_number: form.value.invoice_number,
    customer_number: form.value.customer_number
    });
    
    order.value = response.data;
  } catch (err) {
    if (err.response && err.response.status === 404) {
      error.value = "No order found with that Invoice and Customer number combination.";
    } else {
      error.value = "An error occurred while connecting to the server.";
    }
  } finally {
    loading.value = false;
  }
};

const formatDate = (dateString) => {
  const options = { year: 'numeric', month: 'long', day: 'numeric' };
  return new Date(dateString).toLocaleDateString(undefined, options);
};
</script>

<style scoped>
.tracker-container {
  max-width: 800px;
  margin: 2rem auto;
  font-family: sans-serif;
  padding: 0 1rem;
}

.search-card, .result-card {
  background: white;
  padding: 2rem;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
  margin-bottom: 2rem;
}

.form-group {
  margin-bottom: 1rem;
}

label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: bold;
}

input {
  width: 100%;
  padding: 0.8rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-sizing: border-box; 
}

button {
  width: 100%;
  padding: 1rem;
  background-color: #3b82f6;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1rem;
}

button:disabled {
  background-color: #93c5fd;
}

.error-msg {
  color: #ef4444;
  margin-top: 1rem;
  text-align: center;
}

.header-status {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid #eee;
  padding-bottom: 1rem;
}

.status-badge {
  padding: 0.5rem 1rem;
  border-radius: 20px;
  font-weight: bold;
  color: white;
  background-color: #6b7280; /* Default */
}

.status-badge.ordered { background-color: #3b82f6; }
.status-badge.in-process { background-color: #f59e0b; }
.status-badge.in-route { background-color: #8b5cf6; }
.status-badge.delivered { background-color: #10b981; }

.details-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1rem;
  margin: 1.5rem 0;
}

.products-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 1rem;
}

.products-table th, .products-table td {
  text-align: left;
  padding: 0.75rem;
  border-bottom: 1px solid #eee;
}

.photo-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  gap: 1rem;
  margin-top: 1rem;
}

.photo-card img {
  width: 100%;
  height: auto;
  border-radius: 4px;
}
</style>