<template>
  <div class="lesson-list">
    <div v-for="lesson in lessons" :key="lesson.id" class="lesson-card" :style="{ background: lesson.bg }">
      <div class="icon-wrapper">
        <i :class="['fa', lesson.icon]"></i>
      </div>

      <div class="lesson-info">
        <h2>{{ lesson.topic }}</h2>
        <p class="location">📍 {{ lesson.location }}</p>
        <p class="price">💷 £{{ lesson.price }}</p>
        <p class="spaces" :class="{ soldout: lesson.space === 0 }">
          {{ lesson.space > 0 ? `${lesson.space} spaces left` : 'Sold Out' }}
        </p>

        <button class="add-btn" :disabled="lesson.space === 0" @click="addToCart(lesson)">
          <i class="fa fa-plus-circle"></i>
          {{ lesson.space > 0 ? 'Add to Cart' : 'Unavailable' }}
        </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: ['lessons', 'addToCart'],
  mounted() {
    // pastel background tones
    const pastels = [
      '#f9f7f6', // warm white
      '#f6f3ff', // lavender hint
      '#f3f8ff', // soft blue
      '#fef6f8', // light blush
      '#f9faf3'  // soft sage
    ]
    this.lessons.forEach((lesson, i) => {
      lesson.bg = pastels[i % pastels.length]
    })
  }
}
</script>

<style scoped>
.lesson-list {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
  gap: 28px;
  padding: 1rem 0 3rem;
}

/* === Card Layout === */
.lesson-card {
  border: 1px solid #e8e8e8;
  border-radius: 20px;
  padding: 2rem 1.5rem;
  text-align: center;
  transition: all 0.3s ease;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.04);
}

.lesson-card:hover {
  transform: translateY(-6px);
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.08);
  filter: brightness(1.03);
}

/* === Icon === */
.icon-wrapper {
  background: #393939;
  width: 60px;
  height: 60px;
  margin: 0 auto 1rem;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.icon-wrapper i {
  color: #fff;
  font-size: 22px;
}

/* === Text === */
.lesson-info h2 {
  font-size: 1.25rem;
  margin-bottom: 0.3rem;
  font-weight: 700;
  color: #070707;
}

.lesson-info p {
  margin: 0.25rem 0;
  font-size: 0.95rem;
  color: #333;
}

.price {
  font-weight: 600;
  color: #070707;
}

.spaces {
  margin-top: 0.5rem;
  font-size: 0.9rem;
  color: #555;
  font-weight: 500;
}

.spaces.soldout {
  color: #b91c1c;
  font-weight: 600;
}

/* === Add to Cart Button === */
.add-btn {
  margin-top: 1rem;
  background: #141414;
  color: #fff;
  border: none;
  border-radius: 10px;
  padding: 0.75rem 1.4rem;
  font-weight: 600;
  font-size: 0.95rem;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  width: 100%;
  transition: all 0.25s ease;
}

.add-btn:hover:not(:disabled) {
  background: #1a1a1a;
  transform: translateY(-2px);
  box-shadow: 0 6px 14px rgba(0, 0, 0, 0.15);
}

.add-btn:disabled {
  background: #e6e6e6;
  color: #777;
  cursor: not-allowed;
}
</style>
