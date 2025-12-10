<template>
  <div id="app">

    <!-- ================= HERO SECTION (HIDDEN ON CHECKOUT) ================= -->
    <header class="hero" v-if="!showCart && !checkoutComplete">
      <h1 class="brand">After-Skool</h1>
      <h2 class="tagline">Level Up Your Learning</h2>
      <p class="subtitle">
        Discover after-school classes designed to inspire curiosity, creativity, and growth.
      </p>
      <button @click="scrollToLessons" class="primary-btn">Browse Lessons</button>
    </header>

    <!-- ================= CART BUTTON ================= -->
    <div class="cart-bar" v-if="!checkoutComplete">
      <button class="cart-btn" @click="toggleCart" :disabled="cart.length === 0 && !showCart">
        <i class="fa fa-shopping-cart"></i>
        <span v-if="!showCart">View Cart ({{ cart.length }})</span>
        <span v-else>Back to Lessons</span>
      </button>
    </div>

    <!-- ================= SUCCESS PAGE ================= -->
    <div v-if="checkoutComplete" class="success-page">
      <h1>🎉 Order Successful!</h1>
      <p>Thank you for your order. Your lessons are booked!</p>

      <button class="primary-btn" @click="returnHome">
        Continue Shopping
      </button>
    </div>

    <!-- ================= MAIN CONTENT ================= -->
    <main v-if="!checkoutComplete">

      <!-- ========= LESSONS PAGE ========= -->
      <section v-if="!showCart">
        <div class="controls" ref="lessonsSection">
          <div class="search-controls">
            <input type="text" v-model="searchTerm" @input="onSearchInput"
              placeholder="Search by topic, location, price, or spaces..." class="search-input" />
          </div>

          <div class="sort-controls">
            <label>Sort by:</label>
            <select v-model="sortAttribute">
              <option value="topic">Topic</option>
              <option value="location">Location</option>
              <option value="price">Price</option>
              <option value="space">Spaces</option>
            </select>

            <select v-model="sortOrder">
              <option value="asc">Ascending</option>
              <option value="desc">Descending</option>
            </select>
          </div>
        </div>

        <LessonList :lessons="sortedLessons" :addToCart="addToCart" />
      </section>

      <!-- ========= CHECKOUT PAGE ========= -->
      <section v-else>
        <ShoppingCart :cart="cart" :removeFromCart="removeFromCart" @goBack="showCart = false"
          @order-success="checkoutSuccessHandler" />
      </section>

    </main>

  </div>
</template>

<script>
import LessonList from './components/LessonList.vue'
import ShoppingCart from './components/ShoppingCart.vue'

const ICONS = {
  Math: 'fa-square-root-variable',
  English: 'fa-book-open',
  Science: 'fa-flask-vial',
  Art: 'fa-palette',
  Music: 'fa-music',
  History: 'fa-landmark',
  Geography: 'fa-globe-europe',
  Spanish: 'fa-language',
  Biology: 'fa-dna',
  Chemistry: 'fa-vials'
}

export default {
  components: { LessonList, ShoppingCart },
  data() {
    return {
      showCart: false,
      checkoutComplete: false,
      cart: [],
      lessons: [],
      sortAttribute: 'topic',
      sortOrder: 'asc',
      searchTerm: '',
      searchTimeoutId: null
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
    toggleCart() {
      this.showCart = !this.showCart
    },

    addToCart(lesson) {
      if (lesson.space > 0) {
        lesson.space--
        this.cart.push(lesson)
      }
    },

    removeFromCart(index) {
      const item = this.cart[index]
      item.space++
      this.cart.splice(index, 1)
    },

    scrollToLessons() {
      const section = this.$refs.lessonsSection
      if (section) section.scrollIntoView({ behavior: 'smooth' })
    },

    mapLessons(data) {
      return data.map(lesson => ({
        id: lesson._id,
        topic: lesson.topic,
        location: lesson.location,
        price: lesson.price,
        space: lesson.space,
        icon: ICONS[lesson.topic] || 'fa-book'
      }))
    }, 

    async fetchLessons() {
      try {
        const API_URL = import.meta.env.VITE_API_URL
        const response = await fetch(`${API_URL}/api/lessons`)
        const data = await response.json()
        this.lessons = this.mapLessons(data)
      } catch (err) {
        console.error("❌ Failed to fetch lessons:", err)
      }
    },

    async searchLessons() {
      const term = this.searchTerm.trim()
      const API_URL = import.meta.env.VITE_API_URL

      if (!term) return this.fetchLessons()

      try {
        const response = await fetch(`${API_URL}/api/search?q=${encodeURIComponent(term)}`)
        const data = await response.json()
        this.lessons = this.mapLessons(data)
      } catch (err) {
        console.error("❌ Search failed:", err)
      }
    },

    onSearchInput() {
      clearTimeout(this.searchTimeoutId)
      this.searchTimeoutId = setTimeout(() => this.searchLessons(), 300)
    },

    checkoutSuccessHandler() {
      this.checkoutComplete = true
      this.showCart = false
    },

    returnHome() {
      this.checkoutComplete = false
      this.fetchLessons()
    }
  },

  async mounted() {
    this.fetchLessons()
  }
}
</script>

<style>
/* ================= GLOBAL ================= */
body {
  margin: 0;
  font-family: 'Manrope', sans-serif;
  background-color: #f5f4f2;
  color: #111;
  line-height: 1.6;
}

#app {
  max-width: 1100px;
  margin: auto;
  padding: 2rem;
}

/* ================= HERO ================= */
.hero {
  text-align: center;
  padding: 4rem 1rem 2rem;
}

/* ================= CART BUTTON ================= */
.cart-bar {
  text-align: center;
  margin-bottom: 2rem;
}

.cart-btn {
  background: #545454;
  color: white;
  border: none;
  padding: 0.9rem 1.8rem;
  border-radius: 30px;
  font-weight: 600;
}

.cart-btn:hover:not(:disabled) {
  background: #ffb300;
  color: #111;
}

/* ================= CONTROLS ================= */
.controls {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  margin-bottom: 2rem;
}

.search-input {
  width: 100%;
  padding: 0.75rem 1rem;
  border-radius: 999px;
  border: 1px solid #ddd;
}

/* ================= SUCCESS PAGE ================= */
.success-page {
  text-align: center;
  padding: 4rem 1rem;
}
</style>
