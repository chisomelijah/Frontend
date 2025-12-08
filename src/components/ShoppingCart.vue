<template>
  <button class="back-btn" @click="$emit('goBack')">
    ← Back to Lessons
  </button>

  <div class="checkout-page">
    <h2>Review & Checkout</h2>

    <div class="checkout-container">
      <!-- Left: Cart Summary -->
      <div class="cart-summary">
        <h3>Your Cart</h3>

        <div v-if="cart.length === 0" class="empty-cart">
          <p>Your cart is empty.</p>
        </div>

        <div v-else>
          <div v-for="(lesson, index) in cart" :key="index" class="cart-item">
            <div class="item-info">
              <h4>{{ lesson.topic }}</h4>
              <p class="location">{{ lesson.location }}</p>
              <p class="price">£{{ lesson.price }}</p>
            </div>
            <button class="remove-btn" @click="removeFromCart(index)">✕</button>
          </div>

          <div class="cart-total">
            <span>Total</span>
            <strong>£{{ totalPrice }}</strong>
          </div>
        </div>
      </div>

      <!-- Right: Checkout Form -->
      <div class="checkout-form">
        <h3>Checkout Details</h3>

        <form @submit.prevent="checkout">
          <div class="form-group">
            <label>Name</label>
            <input v-model="name" type="text" placeholder="John Doe" :class="{ invalid: !isValidName && name }" />
          </div>

          <div class="form-group">
            <label>Phone</label>
            <input v-model="phone" type="text" placeholder="Numbers only"
              :class="{ invalid: !isValidPhone && phone }" />
          </div>

          <button class="checkout-btn" type="submit" :disabled="!isValidName || !isValidPhone || cart.length === 0">
            Confirm Order
          </button>
        </form>

        <p v-if="checkoutSuccess" class="success-msg">
          ✅ Order submitted successfully!
        </p>
        <p v-if="checkoutError" class="error-msg">
          ❌ Failed to submit order. Please try again.
        </p>
      </div>
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
    }
  },
  methods: {
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

        const API_URL = import.meta.env.VITE_API_URL
        const response = await fetch(`${API_URL}/api/orders`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(orderData)
        })

        if (!response.ok) throw new Error('Order submission failed')

        this.checkoutSuccess = true
        this.name = ''
        this.phone = ''

        // ✅ Clear the cart AFTER success
        this.cart.splice(0, this.cart.length)

        // ✅ Emit an event to parent (App.vue)
        this.$emit('order-success')

      } catch (err) {
        console.error('❌ Checkout error:', err)
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

.checkout-page h2 {
  text-align: center;
  margin-bottom: 2rem;
  color: #111;
}

.checkout-container {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 2rem;
}

.cart-summary,
.checkout-form {
  background: #fcfcfa;
  border: 1px solid #f0f0f0;
  border-radius: 12px;
  padding: 1.5rem;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.04);
}

.cart-summary h3,
.checkout-form h3 {
  margin-top: 0;
  margin-bottom: 1rem;
  font-weight: 600;
  color: #111;
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
  background: none;
  border: none;
  color: #999;
  font-size: 1.1rem;
  cursor: pointer;
  transition: color 0.2s;
}

.remove-btn:hover {
  color: #e74c3c;
}

.cart-total {
  display: flex;
  justify-content: space-between;
  font-size: 1rem;
  border-top: 1px solid #eee;
  margin-top: 1rem;
  padding-top: 0.75rem;
}

form {
  display: flex;
  flex-direction: column;
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

.checkout-btn {
  padding: 12px;
  border-radius: 10px;
  border: none;
  background-color: #111;
  color: #fff;
  font-weight: 600;
  transition: background 0.3s;
}

.checkout-btn:hover:not(:disabled) {
  background-color: #333;
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
  .checkout-container {
    grid-template-columns: 1fr;
  }
}


.back-btn {
  background: #eee;
  border: none;
  padding: 0.6rem 1.2rem;
  border-radius: 8px;
  font-size: 0.9rem;
  cursor: pointer;
  margin-bottom: 1rem;
}

.back-btn:hover {
  background: #ddd;
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
  .checkout-container {
    grid-template-columns: 1fr;
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
  .checkout-container {
    grid-template-columns: 1fr;
  }
}
</style>
