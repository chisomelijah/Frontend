<template>
  <div id="app">

    <!-- ================= HERO SECTION ================= -->
    <!-- Hidden ONLY when user is viewing the cart -->
    <header class="hero" v-if="!showCart">
      <h1 class="brand">After-Skool</h1>
      <h2 class="tagline">Level Up Your Learning</h2>
      <p class="subtitle">
        Discover after-school classes designed to inspire curiosity, creativity, and growth.
      </p>
      <button @click="scrollToLessons" class="primary-btn">Browse Lessons</button>
    </header>

    <!-- ================= CART BUTTON ================= -->
    <!-- Hidden on checkout page -->
    <div class="cart-bar" v-if="!showCart">
      <button class="cart-btn" @click="toggleCart" :disabled="cart.length === 0">
        <i class="fa fa-shopping-cart"></i>
        View Cart ({{ cart.length }})
      </button>
    </div>

    <!-- ================= MAIN CONTENT ================= -->
    <main>

      <!-- LESSON LIST PAGE -->
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

        <LessonList :lessons="sortedLessons" :addToCart="addToCart" />
      </div>

      <!-- CHECKOUT PAGE -->
      <div v-else>
        <ShoppingCart :cart="cart" :removeFromCart="removeFromCart" @goBack="showCart = false" />
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
      sortAttribute: "topic",
      sortOrder: "asc",
      lessons: [],
      searchTerm: "",
      searchTimeoutId: null
    }
  },

  computed: {
    sortedLessons() {
      return [...this.lessons].sort((a, b) => {
        let x = a[this.sortAttribute];
        let y = b[this.sortAttribute];

        if (typeof x === "string") x = x.toLowerCase();
        if (typeof y === "string") y = y.toLowerCase();

        return this.sortOrder === "asc"
          ? x > y ? 1 : x < y ? -1 : 0
          : x < y ? 1 : x > y ? -1 : 0;
      });
    }
  },

  methods: {
    toggleCart() {
      this.showCart = !this.showCart;
    },

    addToCart(lesson) {
      if (lesson.space > 0) {
        lesson.space--;
        this.cart.push(lesson);
      }
    },

    removeFromCart(index) {
      const item = this.cart[index];
      item.space++;
      this.cart.splice(index, 1);
    },

    scrollToLessons() {
      const section = this.$refs.lessonsSection;
      if (section) section.scrollIntoView({ behavior: "smooth" });
    },

    mapLessons(data) {
      return data.map(lesson => ({
        id: lesson._id,
        topic: lesson.topic,
        location: lesson.location,
        price: lesson.price,
        space: lesson.space,
        icon: ICONS[lesson.topic] || "fa-book"
      }));
    },

    async fetchLessons() {
      try {
        const API_URL = import.meta.env.VITE_API_URL;
        const response = await fetch(`${API_URL}/api/lessons`);
        const data = await response.json();
        this.lessons = this.mapLessons(data);
      } catch (err) {
        console.error("❌ Failed to fetch lessons:", err);
      }
    },

    async searchLessons() {
      const term = this.searchTerm.trim();
      const API_URL = import.meta.env.VITE_API_URL;

      if (!term) {
        this.fetchLessons();
        return;
      }

      try {
        const response = await fetch(`${API_URL}/api/search?q=${encodeURIComponent(term)}`);
        const data = await response.json();
        this.lessons = this.mapLessons(data);
      } catch (err) {
        console.error("❌ Failed to search lessons:", err);
      }
    },

    onSearchInput() {
      if (this.$searchTimeoutId) clearTimeout(this.$searchTimeoutId);
      this.searchTimeoutId = setTimeout(() => this.searchLessons(), 300);
    }
  },

  async mounted() {
    this.fetchLessons();
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Manrope:wght@400;600;700;800&display=swap');
</style>
