<template>
  <div class="checkout-page">

    <!-- BACK BUTTON (NEW) -->
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
      checkoutError: false,
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
    },
  },

  methods: {
    async checkout() {
      this.checkoutSuccess = false;
      this.checkoutError = false;

      try {
        const orderData = {
          name: this.name,
          phone: this.phone,
          lessons: this.cart.map((item) => ({
            lessonId: item.id,
            topic: item.topic,
            price: item.price,
          })),
        };

        const API_URL = import.meta.env.VITE_API_URL;

        const response = await fetch(`${API_URL}/api/orders`, {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(orderData),
        });

        if (!response.ok) throw new Error("Order submission failed");

        this.checkoutSuccess = true;
        this.name = "";
        this.phone = "";

        // Clear cart after checkout
        this.cart.splice(0, this.cart.length);

      } catch (err) {
        console.error("❌ Checkout error:", err);
        this.checkoutError = true;
      }
    },
  },
};
</script>

<style scoped>
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
</style>
