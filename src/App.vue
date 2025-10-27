<template>
  <div id="app">
    <header class="app-header">
      <h1>After-School Classes Booking System</h1>
      <div class="header-right">
        <button v-if="!user" @click="showAuth = true">Login / Signup</button>
        <UserMenu v-else :user="user" @logout="logout" />
      </div>
    </header>

    <!-- Main content -->
    <div v-if="showCart">
      <ShoppingCart :cart="cart" :removeFromCart="removeFromCart" />
    </div>
    <div v-else>
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

    <!-- Auth Modal -->
    <AuthModal v-if="showAuth" @close="showAuth = false" @login="loginUser" />
  </div>
</template>

<script>
import LessonList from './components/LessonList.vue'
import ShoppingCart from './components/ShoppingCart.vue'
import AuthModal from './components/AuthModal.vue'
import UserMenu from './components/UserMenu.vue'

export default {
  components: { LessonList, ShoppingCart, AuthModal, UserMenu },
  data() {
    return {
      user: null,
      showAuth: false,
      showCart: false,
      sortAttribute: 'subject',
      sortOrder: 'asc',
      lessons: [
        { id: 'lesson001', subject: 'Mathematics', location: 'Hendon', price: 100, spaces: 5, icon: 'fa-calculator' },
        { id: 'lesson002', subject: 'English', location: 'Colindale', price: 80, spaces: 5, icon: 'fa-book' },
        { id: 'lesson003', subject: 'Science', location: 'Brent', price: 90, spaces: 5, icon: 'fa-flask' }
      ],
      cart: []
    }
  },
  computed: {
    sortedLessons() {
      return [...this.lessons].sort((a, b) => {
        let x = a[this.sortAttribute], y = b[this.sortAttribute]
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
    loginUser(credentials) {
      // Simulated login
      this.user = { name: credentials.name, email: credentials.email }
      this.showAuth = false
    },
    logout() {
      this.user = null
    }
  }
}
</script>

<style>
.app-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
}

.header-right button {
  padding: 8px 14px;
  border: 1px solid #ddd;
  border-radius: 8px;
  background: #fff;
  cursor: pointer;
}
</style>
