<template>
  <div class="lesson-list">
    <div v-for="lesson in lessons" :key="lesson.id" class="lesson-card">
      <div class="icon-wrapper">
        <i :class="['fa', lesson.icon]"></i>
      </div>

      <div class="lesson-info">
        <h2>{{ lesson.topic }}</h2>
        <p class="location">📍 {{ lesson.location }}</p>
        <p class="price">💷 £{{ lesson.price }}</p>
        <p class="spaces">
          <span v-if="lesson.space > 0">👥 {{ lesson.space }} spaces left</span>
          <span v-else class="sold-out">❌ Sold Out</span>
        </p>

        <button :disabled="lesson.space === 0" @click="addToCart(lesson)">
          {{ lesson.space > 0 ? 'Add to Cart' : 'Unavailable' }}
        </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: ['lessons', 'addToCart']
}
</script>

<style scoped>
.lesson-list {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
  gap: 24px;
  padding: 20px 0;
}

/* Card container */
.lesson-card {
  background: linear-gradient(135deg, #ffffff 0%, #f9f9fb 100%);
  border: 1px solid #f1f1f1;
  border-radius: 20px;
  padding: 1.5rem;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.04);
  text-align: center;
  transition: all 0.25s ease;
  position: relative;
}

.lesson-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 6px 20px rgba(124, 58, 237, 0.15);
}

/* Icon styling */
.icon-wrapper {
  background: linear-gradient(135deg, #ede9fe, #f5f3ff);
  width: 60px;
  height: 60px;
  margin: 0 auto 12px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.icon-wrapper i {
  font-size: 24px;
  color: #7c3aed;
}

/* Text styling */
.lesson-info h2 {
  font-size: 1.25rem;
  margin-bottom: 0.3rem;
  color: #1f1f1f;
  font-weight: 600;
}

.lesson-info p {
  margin: 0.2rem 0;
  font-size: 0.95rem;
  color: #555;
}

.price {
  font-weight: 500;
  color: #333;
}

.spaces {
  margin-top: 0.5rem;
  font-size: 0.9rem;
  color: #666;
}

.sold-out {
  color: #e11d48;
  font-weight: 600;
}

/* Button */
button {
  margin-top: 0.8rem;
  background: linear-gradient(135deg, #7c3aed, #facc15);
  color: white;
  border: none;
  border-radius: 12px;
  padding: 10px 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
  width: 100%;
}

button:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 6px 14px rgba(124, 58, 237, 0.3);
}

button:disabled {
  background: #e5e5e5;
  color: #999;
  cursor: not-allowed;
}
</style>
