<template>
  <div class="user-menu">
    <span>Hello, {{ user.name }}</span>
    <button @click="$emit('logout')">Logout</button>
  </div>
</template>

<script>
export default { props: ['user'] }
</script>

<style scoped>
.user-menu {
  display: flex;
  align-items: center;
  gap: 10px;
}
</style>
