<template>
  <div id="app">
    <!-- Hero Section -->
    <header class="hero">
      <h1 class="brand">After-Skool</h1>
      <h2 class="tagline">Level Up Your Learning</h2>
      <p class="subtitle">
        Discover after-school classes designed to inspire curiosity, creativity, and growth.
      </p>
      <button @click="scrollToLessons" class="primary-btn">Browse Lessons</button>
    </header>

    <!-- Cart Toggle Button -->
    <div class="cart-bar">
      <button class="cart-btn" @click="toggleCart" :disabled="cart.length === 0"
        :title="cart.length === 0 ? 'Your cart is empty' : ''">
        <i class="fa fa-shopping-cart"></i>
        <span v-if="!showCart">View Cart ({{ cart.length }})</span>
        <span v-else>Back to Lessons</span>
      </button>
    </div>


    <!-- Main Content -->
    <main>
      <div v-if="!showCart">
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
<div style="background:red;color:white;padding:4px;">SEARCH AREA TEST</div>

        <LessonList :lessons="sortedLessons" :addToCart="addToCart" />
      </div>

      <div v-else>
        <ShoppingCart :cart="cart" :removeFromCart="removeFromCart" />
      </div>
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
      cart: [],
      sortAttribute: 'topic',
      sortOrder: 'asc',
      lessons: [],
      // 🔹 NEW: search state
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
    toggleCart() {
      this.showCart = !this.showCart
    },
    scrollToLessons() {
      const section = this.$refs.lessonsSection
      if (section) {
        section.scrollIntoView({ behavior: 'smooth', block: 'start' })
      }
    },

    // 🔹 Helper to map raw lesson docs to UI objects
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

    // 🔹 Fetch ALL lessons (used on mount & when search cleared)
    async fetchLessons() {
      try {
        const API_URL = import.meta.env.VITE_API_URL
        const response = await fetch(`${API_URL}/api/lessons`)
        const data = await response.json()
        this.lessons = this.mapLessons(data)
      } catch (err) {
        console.error('❌ Failed to fetch lessons:', err)
      }
    },

    // 🔹 Call backend search endpoint
    async searchLessons() {
      const term = this.searchTerm.trim()
      const API_URL = import.meta.env.VITE_API_URL

      // If search box is empty, restore full list
      if (!term) {
        this.fetchLessons()
        return
      }

      try {
        const response = await fetch(
          `${API_URL}/api/search?q=${encodeURIComponent(term)}`
        )
        const data = await response.json()
        this.lessons = this.mapLessons(data)
      } catch (err) {
        console.error('❌ Failed to search lessons:', err)
      }
    },

    // 🔹 Debounced input handler for "search as you type"
    onSearchInput() {
      if (this.searchTimeoutId) {
        clearTimeout(this.searchTimeoutId)
      }
      this.searchTimeoutId = setTimeout(() => {
        this.searchLessons()
      }, 300)
    }
  },

  async mounted() {
    // use the new helper instead of duplicated fetch logic
    this.fetchLessons()
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Manrope:wght@400;600;700;800&display=swap');

body {
  margin: 0;
  font-family: 'Manrope', sans-serif;
  background-color: #f5f4f2;
  color: #111;
  line-height: 1.6;
}

#app {
  max-width: 1100px;
  margin: 0 auto;
  padding: 2rem;
}

/* === HERO CASCADE === */
.hero {
  text-align: center;
  padding: 5rem 1rem 3rem;
  margin-bottom: 2rem;
}

.brand {
  font-weight: 800;
  font-size: 4rem;
  letter-spacing: -1px;
  margin: 0;
  color: #111;
}

.cart-bar {
  text-align: center;
  margin: 1.5rem 0 2.5rem;
}

.cart-btn {
  background: #545454;
  color: #fff;
  border: none;
  border-radius: 50px;
  padding: 0.9rem 1.8rem;
  font-weight: 600;
  font-size: 1rem;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.6rem;
  cursor: pointer;
  box-shadow: 0 3px 8px rgba(0, 0, 0, 0.1);
  transition: all 0.25s ease;
}

.cart-btn:hover {
  background: #ffb300;
  color: #111;
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.15);
}

.cart-btn i {
  font-size: 1rem;
}

.cart-btn:disabled {
  background: #e5e5e5;
  color: #777;
  cursor: not-allowed;
}

.cart-btn:disabled {
  background: #e5e5e5;
  color: #777;
  cursor: not-allowed;
  box-shadow: none;
}

.cart-btn:not(:disabled) {
  cursor: pointer;
}

.tagline {
  font-size: 1.7rem;
  font-weight: 700;
  color: #333;
  margin: 0.6rem 0;
}

.subtitle {
  font-size: 1rem;
  color: #555;
  max-width: 600px;
  margin: 0 auto 1.5rem;
}

.primary-btn {
  background: #111;
  color: #fff;
  border: none;
  padding: 0.9rem 1.8rem;
  border-radius: 12px;
  font-weight: 600;
  transition: all 0.25s ease;
  box-shadow: 0 3px 10px rgba(0, 0, 0, 0.08);
}

.primary-btn:hover {
  background: #ffb300;
  color: #111;
  box-shadow: 0 5px 16px rgba(0, 0, 0, 0.15);
}

/* === SORT CONTROLS === */
.sort-controls {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-wrap: wrap;
  gap: 1rem;
  padding: 1rem 1.5rem;
  border-radius: 16px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.05);
  margin: 2rem auto;
  max-width: 600px;
}

.sort-controls label {
  font-weight: 600;
  font-size: 1rem;
  color: #222;
}

.sort-controls select {
  border: 1px solid #ddd;
  padding: 0.6rem 1.2rem;
  border-radius: 50px;
  font-size: 1rem;
  background: #f9f9f9;
  color: #333;
  cursor: pointer;
  transition: all 0.2s ease;
  min-width: 150px;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.04);
}

.sort-controls select:hover,
.sort-controls select:focus {
  background: #fff;
  border-color: #ccc;
  box-shadow: 0 3px 8px rgba(0, 0, 0, 0.08);
}

.controls {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  margin: 2rem auto;
  max-width: 700px;
}

.search-controls {
  display: flex;
  justify-content: center;
}

.search-input {
  width: 100%;
  max-width: 700px;
  padding: 0.75rem 1rem;
  border-radius: 999px;
  border: 1px solid #ddd;
  font-size: 1rem;
  background: #f9f9f9;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.04);
}

.search-input:focus {
  outline: none;
  border-color: #ccc;
  background: #fff;
  box-shadow: 0 3px 10px rgba(0, 0, 0, 0.08);
}

/* keep your existing .sort-controls styles as they are */


/* === BUTTON / INTERACTIVES === */
button,
select {
  cursor: pointer;
  transition: all 0.2s ease;
}

/* Subtle responsive polish */
@media (max-width: 600px) {
  .sort-controls {
    flex-direction: column;
    align-items: stretch;
  }

  .sort-controls select {
    width: 100%;
  }
}
</style>
