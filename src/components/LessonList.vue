<template>
    <div class="lesson-list">
        <div v-for="lesson in lessons" :key="lesson.id" class="lesson-card">
            <i :class="['fa', lesson.icon]"></i>
            <h2>{{ lesson.subject }}</h2>
            <p>📍 {{ lesson.location }}</p>
            <p>💷 £{{ lesson.price }}</p>
            <p>👥 {{ lesson.spaces }} spaces available</p>

            <button :disabled="lesson.spaces === 0" @click="addToCart(lesson)">
                Add to Cart
            </button>
        </div>
    </div>
</template>

<script>
export default {
    props: ['lessons', 'addToCart']
}
</script>

<style>
.lesson-list {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 24px;
    padding: 20px 0;
}

.lesson-list button {
    background-color: #1c1c1c;
    color: #fcfcfa;
    border: none;
    padding: 10px 16px;
    border-radius: 8px;
    cursor: pointer;
    font-size: 14px;
    margin-top: 12px;
    transition: background-color 0.3s ease;
}

.lesson-card {
    background-color: #fcfcfa;
    border: 1px solid #efefef;
    border-radius: 16px;
    padding: 20px;
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.04);
    transition: box-shadow 0.25s ease, transform 0.2s ease;
}

.lesson-card:hover {
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
    transform: translateY(-2px);
}

.lesson-card i {
    font-size: 22px;
    color: #444;
    margin-bottom: 6px;
}

.lesson-card h2 {
    font-size: 18px;
    margin-bottom: 0.4rem;
}

.lesson-card p {
    margin: 0.25rem 0;
    color: #555;
}
</style>