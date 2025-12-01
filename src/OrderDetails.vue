<template>
  <div class="result-card">
    <section class="header-status">
      <h2>Order #{{ order.invoice_number }}</h2>
      <span :class="['status-badge', getStatusClass(order.status)]">
        {{ order.status }}
      </span>
    </section>

    <section class="details-grid">
      <div class="detail-item">
        <strong>Order Date:</strong>
        <span>{{ formatDate(order.order_date) }}</span>
      </div>
      <div class="detail-item">
        <strong>Customer:</strong>
        <span>{{ order.customer.name }}</span>
      </div>
      <div class="detail-item">
        <strong>Total:</strong>
        <span class="total-amount">${{ order.total_amount || '0.00' }}</span>
      </div>
      <div class="detail-item full-width">
        <strong>Delivery Address:</strong>
        <span>{{ order.delivery_address }}</span>
      </div>
    </section>

    <h3 class="section-title">Products Included</h3>
    <table class="products-table">
      <thead>
        <tr>
          <th>Product</th>
          <th>Qty.</th>
          <th>Unit Price</th>
          <th>Subtotal</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="product in order.products" :key="product.id">
          <td>{{ product.title }}</td>
          <td>{{ product.pivot?.quantity || product.quantity }}</td>
          <td>${{ product.pivot?.unit_price || product.unit_price }}</td>
          <td>${{ ((product.pivot?.quantity || 0) * (product.pivot?.unit_price || 0)).toFixed(2) }}</td>
        </tr>
      </tbody>
    </table>

    <section v-if="order.photos && order.photos.length > 0" class="photos-section">
      <h3 class="section-title">Delivery Evidence</h3>
      <div class="photo-grid">
        <div v-for="photo in order.photos" :key="photo.id" class="photo-card">
          <img :src="photo.photo_url || photo.photo_path" alt="Evidence" loading="lazy" />
          <div class="photo-info">
            <span class="photo-type">{{ photo.photo_type }}</span>
            <small>{{ formatDate(photo.uploaded_at) }}</small>
          </div>
        </div>
      </div>
    </section>
    <div v-else class="no-photos">
        <p>No photo evidence available yet.</p>
    </div>
  </div>
</template>

<script setup>
const props = defineProps({
  order: {
    type: Object,
    required: true
  },
  formatDate: {
    type: Function,
    required: true
  },
  getStatusClass: {
    type: Function,
    required: true
  }
});
</script>

<style scoped>
.result-card { background: white; padding: 2rem; border-radius: 8px; border: 1px solid #e2e8f0; }
.header-status { display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #e2e8f0; padding-bottom: 1rem; margin-bottom: 1.5rem; gap: 10px; }
h2 { margin: 0; font-size: 1.5rem; color: #1a202c; }
.status-badge { padding: 0.4rem 0.8rem; border-radius: 12px; font-weight: 600; color: white; font-size: 0.8rem; text-transform: uppercase; }
.ordered { background-color: #4299e1; }
.in-process { background-color: #ecc94b; }
.in-route { background-color: #805ad5; }
.delivered { background-color: #48bb78; }
.details-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 1.5rem; margin-bottom: 2rem; }
.detail-item strong { display: block; color: #718096; font-size: 0.85rem; margin-bottom: 0.2rem; font-weight: 600; }
.detail-item span { font-size: 1rem; color: #2d3748; font-weight: 500; }
.detail-item.full-width { grid-column: 1 / -1; }
.total-amount { font-size: 1.25rem !important; font-weight: 700 !important; color: #2d3748 !important; }
.section-title { font-size: 1.25rem; color: #1a202c; margin-top: 2rem; margin-bottom: 1rem; border-left: 4px solid #3182ce; padding-left: 0.75rem; }
.products-table { width: 100%; border-collapse: collapse; margin-top: 1rem; }
.products-table th, .products-table td { text-align: left; padding: 0.75rem 0.5rem; border-bottom: 1px solid #edf2f7; }
.products-table th { background-color: #f7fafc; font-weight: 600; font-size: 0.8rem; color: #4a5568; text-transform: uppercase; letter-spacing: 0.05em; }
.products-table tr:last-child td { border-bottom: none; }
.photo-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(180px, 1fr)); gap: 1rem; margin-top: 0.5rem; }
.photo-card { border: 1px solid #e2e8f0; border-radius: 6px; background: #ffffff; overflow: hidden; display: flex; flex-direction: column; }
.photo-card img { width: 100%; height: 120px; object-fit: cover; transition: transform 0.2s; }
.photo-card img:hover { transform: scale(1.03); }
.photo-info { padding: 0.5rem; display: flex; flex-direction: column; }
.photo-type { background-color: #edf2f7; color: #4a5568; padding: 2px 6px; border-radius: 3px; font-size: 0.7rem; font-weight: bold; text-transform: uppercase; align-self: flex-start; margin-bottom: 4px; }
.no-photos { margin-top: 1.5rem; padding: 1rem; background: #f7fafc; border-radius: 4px; text-align: center; color: #718096; font-style: italic; }
</style>