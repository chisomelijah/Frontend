<template>
  <div id="app">
    <h1>After-Skool</h1>

    <button :disabled="cart.length === 0" @click="toggleCart" style="margin: .5rem;">
      {{ showCart ? 'Back to Lessons' : `View Cart (${cart.length})` }}
    </button>

    <button @click="showAuth = true">Login / Signup</button>

    <div v-if="!showCart">
      <!-- Sort controls + lessons -->
      <div class="sort-controls">
        <label>Sort by:</label>
        <select v-model="sortAttribute">
          <option value="subject">Subject</option>
          <option value="location">Location</option>
          <option value="price">Price</option>
          <option value="spaces">Spaces</option>
        </select>
        <select v-model="sortOrder">
          <option value="asc">Ascending</option>
          <option value="desc">Descending</option>
        </select>
      </div>

      <LessonList :lessons="sortedLessons" :addToCart="addToCart" />
    </div>

    <div v-else>
      <ShoppingCart :cart="cart" :removeFromCart="removeFromCart" />
    </div>
  </div>
  <AuthModal v-if="showAuth" @close="showAuth = false" />
</template>

<script>
import LessonList from './components/LessonList.vue'
import ShoppingCart from './components/ShoppingCart.vue'
import AuthModal from './components/AuthModal.vue'

export default {
  components: { LessonList, ShoppingCart, AuthModal },
  data() {
    return {
      showCart: false,
      showAuth: false,
      cart: [],
      sortAttribute: 'subject',
      sortOrder: 'asc',
      lessons: [

        { id: 'lesson001', subject: 'Mathematics', location: 'Hendon', price: 100, spaces: 5, icon: 'fa-calculator' },
        { id: 'lesson002', subject: 'English Literature', location: 'Colindale', price: 85, spaces: 3, icon: 'fa-book-open' },
        { id: 'lesson003', subject: 'Computer Science', location: 'Brent', price: 95, spaces: 4, icon: 'fa-laptop-code' },
        { id: 'lesson004', subject: 'Physics', location: 'Barnet', price: 110, spaces: 6, icon: 'fa-atom' },
        { id: 'lesson005', subject: 'Biology', location: 'Kilburn', price: 90, spaces: 2, icon: 'fa-dna' },
        { id: 'lesson006', subject: 'Networking Fundamentals', location: 'Hendon', price: 120, spaces: 5, icon: 'fa-network-wired' },
        { id: 'lesson007', subject: 'Chemistry', location: 'Colindale', price: 100, spaces: 4, icon: 'fa-vials' },
        { id: 'lesson008', subject: 'Graphic Design', location: 'Brent', price: 80, spaces: 3, icon: 'fa-paint-brush' },
        { id: 'lesson009', subject: 'Product Design', location: 'Brondesbury', price: 105, spaces: 6, icon: 'fa-cube' },
        { id: 'lesson010', subject: 'Web Development', location: 'Golders Green', price: 95, spaces: 5, icon: 'fa-code' }

      ]
    }

  },
  computed: {
    sortedLessons() {
      return [...this.lessons].sort((a, b) => {
        let x = a[this.sortAttribute]
        let y = b[this.sortAttribute]
        if (typeof x === 'string') x = x.toLowerCase()
        if (typeof y === 'string') y = y.toLowerCase()
        return this.sortOrder === 'asc'
          ? x > y ? 1 : x < y ? -1 : 0
          : x < y ? 1 : x > y ? -1 : 0
      })
    }
  },
  methods: {
    addToCart(lesson) {
      if (lesson.spaces > 0) {
        lesson.spaces--
        this.cart.push(lesson)
      }
    },
    removeFromCart(index) {
      const item = this.cart[index]
      item.spaces++
      this.cart.splice(index, 1)
    },
    toggleCart() {
      this.showCart = !this.showCart
    }
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Manrope:wght@400;500;600;700&display=swap');

body {
  margin: 0;
  font-family: 'Manrope', sans-serif;
  background-color: #f9f9f8;
  /* soft white */
  color: #222;
  /* deep neutral text */
  line-height: 1.6;
}

#app {
  max-width: 1100px;
  margin: 0 auto;
  padding: 2rem;
}

h1,
h2,
h3 {
  font-weight: 700;
  color: #111;
}

button,
select,
input {
  font-family: inherit;
  font-weight: 500;
  background: #fff;
  color: #222;
  border: 1px solid #d2d2d2;
  border-radius: 8px;
  padding: 8px 14px;
  transition: all 0.2s ease;
}

button:hover:not(:disabled),
select:hover,
input:focus {
  border-color: #cfcfcf;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.06);
}

button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.sort-controls {
  display: flex;
  gap: 0.75rem;
  flex-wrap: wrap;
  margin: 1.5rem 0;
}

hr {
  border: none;
  border-top: 1px solid #eee;
  margin: 1.5rem 0;
}
</style>