<template>
  <div class="tracker-container">
    <h1 class="main-title">Order Tracking</h1>

    <div class="search-card">
      <form @submit.prevent="fetchOrder" class="form-grid">
        <div class="form-group">
          <label for="invoice">Invoice #</label>
          <input 
            v-model="form.invoice_number" 
            type="text" 
            id="invoice"
            placeholder="INV-2023-001" 
            required 
          />
        </div>

        <div class="form-group">
          <label for="customer">Customer Number</label>
          <input 
            v-model="form.customer_number" 
            type="text" 
            id="customer"
            placeholder="CUST-9988" 
            required 
          />
        </div>

        <button type="submit" :disabled="loading" class="track-button">
          {{ loading ? 'Searching...' : 'Track Order' }}
        </button>
      </form>
      
      <p v-if="error" class="error-msg">{{ error }}</p>
    </div>

    <OrderDetails 
      v-if="order" 
      :order="order" 
      :formatDate="formatDate" 
      :getStatusClass="getStatusClass" 
    />
  </div>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';
import OrderDetails from './OrderDetails.vue'; 
import { formatDate, getStatusClass } from './utils/formatUtils'; 

const form = ref({
  invoice_number: '',
  customer_number: ''
});

const order = ref(null);
const loading = ref(false);
const error = ref(null);

const fetchOrder = async () => {
  loading.value = true;
  error.value = null;
  order.value = null;

  try {
    const validationResponse = await axios.get(
      `http://127.0.0.1:8000/api/orders/invoice/${form.value.invoice_number}/customer/${form.value.customer_number}`
    );

    const orderData = validationResponse.data.data;
    
    if (!orderData || validationResponse.data.success === false) {
      throw new Error('NOT_FOUND');
    }

    order.value = orderData;

  } catch (err) {
    if (err.message === 'NOT_FOUND' || err.response?.status === 404 || err.response?.status === 401) {
      error.value = "We could not find an order matching that Invoice Number and Customer.";
    } else {
      error.value = "An error occurred while fetching information. Please try again.";
    }
  } finally {
    loading.value = false;
  }
};
</script>

<style scoped>
.tracker-container { max-width: 700px; margin: 3rem auto; padding: 0 1.5rem; font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif; }
.main-title { text-align: center; font-size: 2rem; font-weight: 700; color: #1a202c; margin-bottom: 2.5rem; }
.search-card { background: white; padding: 2rem; border-radius: 8px; border: 1px solid #e2e8f0; margin-bottom: 2rem; }
.form-grid { display: grid; grid-template-columns: 1fr 1fr 150px; gap: 1rem; align-items: end; }
.form-group { margin-bottom: 0; }
label { display: block; margin-bottom: 0.4rem; font-weight: 500; color: #4a5568; font-size: 0.875rem; }
input { width: 100%; padding: 0.75rem; border: 1px solid #cbd5e0; border-radius: 4px; box-sizing: border-box; font-size: 0.95rem; transition: border-color 0.15s; }
input:focus { outline: none; border-color: #3182ce; }
.track-button { padding: 0.75rem 1rem; background-color: #3182ce; color: white; border: none; border-radius: 4px; cursor: pointer; font-size: 1rem; font-weight: 600; transition: background-color 0.15s; }
.track-button:hover:not(:disabled) { background-color: #2c5282; }
.track-button:disabled { background-color: #90cdf4; cursor: not-allowed; }
.error-msg { color: #e53e3e; background-color: #fff5f5; padding: 0.75rem; border-radius: 4px; margin-top: 1.5rem; text-align: center; border: 1px solid #fed7d7; font-size: 0.95rem; }
@media (max-width: 600px) { .form-grid { grid-template-columns: 1fr; gap: 1.25rem; } }
</style>