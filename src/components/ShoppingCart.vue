<template>
  <div class="checkout-page">

    <!-- BACK BUTTON -->
    <button class="back-btn" @click="$emit('goBack')">
      ← Back to Lessons
    </button>

    <h2>Review & Checkout</h2>

    <div class="checkout-container">

      <!-- LEFT: CART SUMMARY -->
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

      <!-- RIGHT: CHECKOUT FORM -->
      <div class="checkout-form">
        <h3>Checkout Details</h3>

        <form @submit.prevent="checkout">
          <div class="form-group">
            <label>Name</label>
            <input v-model="name" type="text" placeholder="John Doe" :class="{ invalid: name && !isValidName }" />
          </div>

          <div class="form-group">
            <label>Phone</label>
            <input v-model="phone" type="text" placeholder="Numbers only"
              :class="{ invalid: phone && !isValidPhone }" />
          </div>

          <button class="checkout-btn" type="submit" :disabled="!isValidName || !isValidPhone || cart.length === 0">
            Confirm Order
          </button>
        </form>

        <!-- SUCCESS & ERROR MESSAGES -->
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
  props: ["cart", "removeFromCart"],

  data() {
    return {
      name: "",
      phone: "",
      checkoutSuccess: false,
      checkoutError: false
    };
  },

  computed: {
    isValidName() {
      return /^[A-Za-z\s]+$/.test(this.name);
    },
    isValidPhone() {
      return /^[0-9]+$/.test(this.phone);
    },
    totalPrice() {
      return this.cart.reduce((sum, item) => sum + item.price, 0);
    }
  },

  methods: {
    async checkout() {
      this.checkoutSuccess = false;
      this.checkoutError = false;

      try {
        const orderData = {
          name: this.name,
          phone: this.phone,
          lessons: this.cart.map(item => ({
            lessonId: item.id,
            topic: item.topic,
            price: item.price
          }))
        };

        const API_URL = import.meta.env.VITE_API_URL;

        const response = await fetch(`${API_URL}/api/orders`, {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(orderData)
        });

        if (!response.ok) throw new Error("Order submission failed");

        // UI feedback inside checkout
        this.checkoutSuccess = true;
        this.name = "";
        this.phone = "";

        // Clear cart
        this.cart.splice(0, this.cart.length);

        // Tell App.vue the order completed
        this.$emit("order-success");

      } catch (err) {
        console.error("❌ Checkout error:", err);
        this.checkoutError = true;
      }
    }
  }
};
</script>

<style scoped>
.checkout-page {
  background: #fff;
  padding: 2rem;
  border-radius: 16px;
  max-width: 950px;
  margin: auto;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.08);
}

.back-btn {
  background: #eee;
  border: none;
  padding: 0.6rem 1.2rem;
  border-radius: 8px;
  cursor: pointer;
  margin-bottom: 1rem;
  font-weight: 600;
}

.back-btn:hover {
  background: #ddd;
}

.checkout-container {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 2rem;
}

.cart-summary,
.checkout-form {
  background: #fafafa;
  padding: 1.5rem;
  border-radius: 12px;
  border: 1px solid #ececec;
}

.cart-item {
  display: flex;
  justify-content: space-between;
  background: #fff;
  padding: 0.75rem 1rem;
  border-radius: 10px;
  margin-bottom: 0.7rem;
  border: 1px solid #eee;
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
  font-weight: 600;
}

.remove-btn {
  background: none;
  border: none;
  color: #888;
  font-size: 1.2rem;
  cursor: pointer;
}

.remove-btn:hover {
  color: #e74c3c;
}

.cart-total {
  display: flex;
  justify-content: space-between;
  margin-top: 1rem;
  font-weight: 600;
}

form {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

input {
  padding: 10px;
  border-radius: 8px;
  border: 1px solid #ddd;
}

input.invalid {
  border-color: #ff6b6b;
  background: #fff5f5;
}

.checkout-btn {
  background: #111;
  color: #fff;
  padding: 12px;
  border-radius: 10px;
  border: none;
  font-weight: 600;
}

.checkout-btn:hover:not(:disabled) {
  background: #333;
}

.success-msg {
  margin-top: 1rem;
  background: #e8f8e8;
  padding: 10px;
  border-radius: 8px;
  color: #2e7d32;
  text-align: center;
  font-weight: 600;
}

.error-msg {
  margin-top: 1rem;
  background: #fdeaea;
  padding: 10px;
  border-radius: 8px;
  color: #b71c1c;
  text-align: center;
  font-weight: 600;
}

/* Mobile */
@media (max-width: 768px) {
  .checkout-container {
    grid-template-columns: 1fr;
  }
}
</style>
