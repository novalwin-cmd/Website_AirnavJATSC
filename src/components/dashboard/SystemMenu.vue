<script setup>
defineProps({
  systems: Object
})

defineEmits(['select'])
</script>

<template>
  <div class="dashboard-grid">
    <div
      v-for="(system, key) in systems"
      :key="key"
      class="card"
      @click="$emit('select', key)"
    >
      <div class="card-icon">{{ system.icon }}</div>
      <div class="card-title">{{ system.name }}</div>
      <div class="card-subtitle">{{ system.subtitle }}</div>
    </div>
  </div>
</template>

<style scoped>
.dashboard-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 25px;
  margin-bottom: 30px;
}

.card {
  background: linear-gradient(135deg, rgba(26, 35, 50, 0.8) 0%, rgba(15, 20, 25, 0.8) 100%);
  border: 1px solid rgba(76, 219, 189, 0.2);
  border-radius: 16px;
  padding: 24px;
  cursor: pointer;
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
}

.card::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 2px;
  background: linear-gradient(90deg, transparent, #4cdbbd, transparent);
}

.card:hover {
  border-color: rgba(76, 219, 189, 0.5);
  transform: translateY(-5px);
  box-shadow: 0 15px 40px rgba(76, 219, 189, 0.15);
}

.card-icon {
  font-size: 32px;
  margin-bottom: 12px;
}

.card-title {
  font-size: 18px;
  font-weight: 600;
  color: #ffffff;
  margin-bottom: 8px;
}

.card-subtitle {
  font-size: 12px;
  color: #a0aec0;
}
</style>
