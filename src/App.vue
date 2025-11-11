<template>
  <div id="app">
    <h1>After-Skool</h1>

    <button @click="toggleCart" :class="{ disabled: cart.length === 0 && !showCart }" style="margin: .5rem;">
      {{ showCart ? 'Back to Lessons' : `View Cart (${cart.length})` }}
    </button>


    <div v-if="!showCart">
      <!-- Sort controls + lessons -->
      <div class="sort-controls">
        <label>Sort by:</label>
        <select v-model="sortAttribute">
          <option value="topic">Topic</option>
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
      <ShoppingCart :cart="cart" :removeFromCart="removeFromCart" @order-success="handleOrderSuccess" />

    </div>
  </div>
</template>

<script>
import LessonList from './components/LessonList.vue'
import ShoppingCart from './components/ShoppingCart.vue'

export default {
  components: { LessonList, ShoppingCart },

  data() {
    return {
      showCart: false,
      cart: [],
      sortAttribute: 'topic',
      sortOrder: 'asc',
      lessons: [] // fetched from backend
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
    },
    handleOrderSuccess() {
      console.log('✅ Order success — returning to lessons...')
      setTimeout(() => {
        this.showCart = false // switch back to lessons
      }, 2000)
    },

    async mounted() {
      try {
        const response = await fetch(`${import.meta.env.VITE_API_URL}/lessons`)
        const data = await response.json()

        const icons = {
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

        this.lessons = data.map(lesson => ({
          id: lesson._id,
          topic: lesson.topic,
          location: lesson.location,
          price: lesson.price,
          spaces: lesson.space, // 🔥 consistent with backend
          icon: icons[lesson.topic] || 'fa-book' // fallback icon
        }))

        console.log('✅ Lessons loaded:', this.lessons)
      } catch (err) {
        console.error('❌ Failed to fetch lessons:', err)
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
  color: #222;
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

button.disabled {
  opacity: 0.5;
  cursor: not-allowed;
  pointer-events: none;
}

button.disabled:hover {
  border-color: #d2d2d2;
  box-shadow: none;
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
