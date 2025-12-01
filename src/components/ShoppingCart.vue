<template>
  <div class="checkout-page">
    <div class="checkout-header">
      <div>
        <p class="eyebrow">Checkout</p>
        <h2>Review and confirm</h2>
        <p class="subtext">Double-check your cart and contact details before placing the order.</p>
      </div>
      <div class="summary-pill">
        <span>{{ itemCount }} {{ itemCount === 1 ? 'item' : 'items' }}</span>
        <strong>{{ formatPrice(totalPrice) }}</strong>
      </div>
    </div>

    <div class="checkout-grid">
      <section class="panel cart-summary">
        <header class="panel-header">
          <div>
            <p class="eyebrow">Order summary</p>
            <h3>Your cart</h3>
          </div>
        </header>

        <div v-if="cart.length === 0" class="empty-cart">
          <p>Your cart is empty.</p>
        </div>

        <div v-else>
          <div v-for="(lesson, index) in cart" :key="index" class="cart-item">
            <div class="item-info">
              <h4>{{ lesson.topic }}</h4>
              <p class="location">{{ lesson.location }}</p>
              <p class="price">{{ formatPrice(lesson.price) }}</p>
            </div>
            <button class="remove-btn" @click="removeFromCart(index)" aria-label="Remove from cart">
              Remove
            </button>
          </div>

          <div class="cart-total">
            <span>Total</span>
            <strong>{{ formatPrice(totalPrice) }}</strong>
          </div>
        </div>
      </section>

      <section class="panel checkout-form">
        <header class="panel-header">
          <div>
            <p class="eyebrow">Details</p>
            <h3>Contact info</h3>
          </div>
        </header>

        <form @submit.prevent="checkout" class="form-grid">
          <div class="form-group">
            <label for="name">Name</label>
            <input id="name" v-model="name" type="text" placeholder="John Doe"
              :class="{ invalid: !isValidName && name }" />
            <small v-if="!isValidName && name" class="error-note">Only letters and spaces are allowed.</small>
          </div>

          <div class="form-group">
            <label for="phone">Phone</label>
            <input id="phone" v-model="phone" type="text" placeholder="Numbers only"
              :class="{ invalid: !isValidPhone && phone }" />
            <small v-if="!isValidPhone && phone" class="error-note">Use digits only.</small>
          </div>

          <button class="checkout-btn" type="submit"
            :disabled="!isValidName || !isValidPhone || cart.length === 0">
            Confirm order
          </button>
        </form>

        <p v-if="checkoutSuccess" class="success-msg">
          Order submitted successfully!
        </p>
        <p v-if="checkoutError" class="error-msg">
          Failed to submit order. Please try again.
        </p>
      </section>
    </div>
  </div>
</template>

<script>
export default {
  props: ['cart', 'removeFromCart'],
  data() {
    return {
      name: '',
      phone: '',
      checkoutSuccess: false,
      checkoutError: false
    }
  },
  computed: {
    isValidName() {
      return /^[A-Za-z\s]+$/.test(this.name)
    },
    isValidPhone() {
      return /^[0-9]+$/.test(this.phone)
    },
    totalPrice() {
      return this.cart.reduce((sum, item) => sum + item.price, 0)
    },
    itemCount() {
      return this.cart.length
    }
  },
  methods: {
    getApiBase() {
      const base = import.meta.env.VITE_API_URL
      if (!base) {
        throw new Error('VITE_API_URL is not set; cannot reach backend.')
      }
      return base.replace(/\/$/, '')
    },
    formatPrice(value) {
      const number = Number(value) || 0
      return `$${number.toFixed(2)}`
    },
    async checkout() {
      this.checkoutSuccess = false
      this.checkoutError = false

      try {
        const orderData = {
          name: this.name,
          phone: this.phone,
          lessons: this.cart.map(item => ({
            lessonId: item.id,
            topic: item.topic,
            price: item.price
          }))
        }

        const API_URL = this.getApiBase()
        const response = await fetch(`${API_URL}/api/orders`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(orderData)
        })

        if (!response.ok) throw new Error('Order submission failed')

        this.checkoutSuccess = true
        this.name = ''
        this.phone = ''

        // Clear the cart AFTER success
        this.cart.splice(0, this.cart.length)

        // Emit an event to parent (App.vue)
        this.$emit('order-success')

      } catch (err) {
        console.error('Checkout error:', err)
        this.checkoutError = true
      }
    }

  }
}
</script>

<style scoped>
.checkout-page {
  background: #fff;
  border-radius: 16px;
  padding: 2rem;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
  margin: 2rem auto;
  max-width: 900px;
}

.checkout-header {
  display: flex;
  justify-content: space-between;
  gap: 1rem;
  align-items: flex-start;
  margin-bottom: 1.75rem;
}

.checkout-header h2 {
  margin: 0.2rem 0 0.35rem;
}

.eyebrow {
  text-transform: uppercase;
  letter-spacing: 0.08em;
  font-size: 0.75rem;
  color: #888;
  margin: 0;
}

.subtext {
  margin: 0;
  color: #666;
}

.summary-pill {
  background: #111;
  color: #fff;
  border-radius: 14px;
  padding: 0.9rem 1.2rem;
  min-width: 160px;
  text-align: right;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.12);
}

.summary-pill span {
  display: block;
  font-size: 0.9rem;
  opacity: 0.85;
}

.summary-pill strong {
  font-size: 1.2rem;
}

.checkout-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 2rem;
}

.panel {
  background: #fcfcfa;
  border: 1px solid #f0f0f0;
  border-radius: 12px;
  padding: 1.5rem;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.04);
}

.panel-header h3 {
  margin: 0.2rem 0 0.5rem;
}

.panel-header {
  margin-bottom: 1rem;
}

.cart-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: #fff;
  border: 1px solid #ececec;
  border-radius: 10px;
  padding: 0.75rem 1rem;
  margin-bottom: 0.75rem;
}

.item-info h4 {
  margin: 0;
  font-size: 1rem;
  font-weight: 600;
}

.location {
  font-size: 0.9rem;
  color: #777;
}

.price {
  font-size: 0.9rem;
  color: #333;
}

.remove-btn {
  background: #f5f5f5;
  border: 1px solid #e6e6e6;
  color: #333;
  font-size: 0.85rem;
  padding: 0.4rem 0.75rem;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.remove-btn:hover {
  background: #ffecec;
  border-color: #f0b1b1;
  color: #b91c1c;
}

.cart-total {
  display: flex;
  justify-content: space-between;
  font-size: 1rem;
  border-top: 1px solid #eee;
  margin-top: 1rem;
  padding-top: 0.75rem;
}

.form-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 1rem;
}

.form-group {
  display: flex;
  flex-direction: column;
}

label {
  margin-bottom: 0.3rem;
  font-weight: 500;
  color: #444;
}

input {
  padding: 10px 12px;
  border: 1px solid #ddd;
  border-radius: 8px;
  background: #fff;
  transition: border-color 0.2s, box-shadow 0.2s;
}

input:focus {
  border-color: #bbb;
  box-shadow: 0 0 0 2px rgba(0, 0, 0, 0.03);
  outline: none;
}

.invalid {
  border-color: #ff6b6b;
  background-color: #fff7f7;
}

.error-note {
  color: #b91c1c;
  font-size: 0.85rem;
  margin-top: 0.25rem;
}

.checkout-btn {
  padding: 12px;
  border-radius: 10px;
  border: none;
  background-color: #111;
  color: #fff;
  font-weight: 600;
  transition: background 0.3s;
  margin-top: 0.5rem;
}

.checkout-btn:hover:not(:disabled) {
  background-color: #333;
}

.checkout-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.success-msg {
  color: #2e7d32;
  background: #eaf6ea;
  border-radius: 8px;
  padding: 8px 12px;
  margin-top: 12px;
  font-weight: 500;
  text-align: center;
  animation: fadeIn 0.6s ease-in-out;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(4px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.error-msg {
  color: #b71c1c;
  background: #fdecea;
  border-radius: 8px;
  padding: 8px 12px;
  margin-top: 12px;
  font-weight: 500;
  text-align: center;
}

/* Responsive */
@media (max-width: 768px) {
  .checkout-grid {
    grid-template-columns: 1fr;
  }

  .checkout-header {
    flex-direction: column;
    align-items: flex-start;
  }

  .summary-pill {
    width: 100%;
    text-align: left;
  }
}
</style>
