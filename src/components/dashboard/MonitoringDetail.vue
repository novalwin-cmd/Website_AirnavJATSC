<script setup>
defineProps({
  system: Object,
  user: String
})

defineEmits(['acb-control'])

const canControl = (user) => user === 'admin'
</script>

<template>
  <div v-if="system">
    <div class="monitoring-header">
      <div>
        <h2 class="detail-title">{{ system.name }}</h2>
        <p class="page-description">{{ system.description }}</p>
      </div>
    </div>

    <!-- Metrics Grid -->
    <div class="metrics-grid">
      <div v-for="(metric, index) in system.metrics" :key="index" class="metric-card">
        <div class="metric-icon">{{ metric.icon }}</div>
        <div class="metric-label">{{ metric.label }}</div>
        <div class="metric-value" :style="{ color: metric.color }">{{ metric.value }}</div>
        <div class="metric-chart">
          <div class="metric-line" :style="{ transform: `scaleX(${metric.line / 100})`, background: metric.color }"></div>
        </div>
      </div>
    </div>

    <!-- Controls Section for Power Meter Panels -->
    <div v-if="system.controls" class="panel-section">
      <div class="panel-title">{{ system.name }} - Kontrol ACB</div>
      <div class="panel-info">ACB Status: <strong style="color: #4cdbbd;">{{ system.acbStatus }}</strong></div>
      <div class="panel-controls">
        <button
          v-if="canControl(user)"
          class="control-btn btn-open"
          @click="$emit('acb-control', 'open')"
        >
          Open
        </button>
        <button
          v-if="canControl(user)"
          class="control-btn btn-trip"
          @click="$emit('acb-control', 'trip')"
        >
          Trip
        </button>
        <button
          v-if="canControl(user)"
          class="control-btn btn-close"
          @click="$emit('acb-control', 'close')"
        >
          Close
        </button>
        <div v-if="!canControl(user)" class="permission-notice">
          Only admin can control ACB
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.monitoring-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30px;
}

.detail-title {
  font-size: 28px;
  font-weight: 600;
  color: #ffffff;
  margin-bottom: 8px;
}

.page-description {
  color: #a0aec0;
  font-size: 14px;
}

.metrics-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 20px;
  margin-bottom: 30px;
}

.metric-card {
  background: linear-gradient(135deg, rgba(26, 35, 50, 0.8) 0%, rgba(15, 20, 25, 0.8) 100%);
  border: 1px solid rgba(76, 219, 189, 0.2);
  border-radius: 12px;
  padding: 20px;
  transition: all 0.3s ease;
}

.metric-card:hover {
  border-color: rgba(76, 219, 189, 0.5);
  box-shadow: 0 5px 15px rgba(76, 219, 189, 0.1);
}

.metric-icon {
  font-size: 20px;
  margin-bottom: 8px;
}

.metric-label {
  font-size: 12px;
  color: #a0aec0;
  margin-bottom: 8px;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.metric-value {
  font-size: 24px;
  font-weight: 600;
  color: #4cdbbd;
  margin-bottom: 8px;
}

.metric-chart {
  height: 40px;
  background: rgba(76, 219, 189, 0.1);
  border-radius: 4px;
  position: relative;
  overflow: hidden;
}

.metric-line {
  height: 100%;
  opacity: 0.6;
  border-radius: 4px;
  transform-origin: left;
  transition: transform 0.3s ease;
}

.panel-section {
  background: linear-gradient(135deg, rgba(26, 35, 50, 0.8) 0%, rgba(15, 20, 25, 0.8) 100%);
  border: 1px solid rgba(76, 219, 189, 0.2);
  border-radius: 16px;
  padding: 30px;
  margin-bottom: 25px;
}

.panel-title {
  font-size: 20px;
  font-weight: 600;
  color: #ffffff;
  margin-bottom: 12px;
}

.panel-info {
  font-size: 13px;
  color: #a0aec0;
  margin-bottom: 20px;
}

.panel-controls {
  display: flex;
  gap: 12px;
  justify-content: flex-start;
  flex-wrap: wrap;
  align-items: center;
}

.control-btn {
  padding: 10px 20px;
  border: none;
  border-radius: 6px;
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
}

.btn-open {
  background: rgba(76, 219, 189, 0.2);
  color: #4cdbbd;
  border: 1px solid #4cdbbd;
}

.btn-open:hover {
  background: rgba(76, 219, 189, 0.3);
  box-shadow: 0 5px 15px rgba(76, 219, 189, 0.2);
}

.btn-close {
  background: rgba(239, 68, 68, 0.2);
  color: #ef4444;
  border: 1px solid #ef4444;
}

.btn-close:hover {
  background: rgba(239, 68, 68, 0.3);
  box-shadow: 0 5px 15px rgba(239, 68, 68, 0.2);
}

.btn-trip {
  background: rgba(249, 115, 22, 0.2);
  color: #f97316;
  border: 1px solid #f97316;
}

.btn-trip:hover {
  background: rgba(249, 115, 22, 0.3);
  box-shadow: 0 5px 15px rgba(249, 115, 22, 0.2);
}

.permission-notice {
  color: #f97316;
  font-size: 13px;
  font-style: italic;
}

@media (max-width: 768px) {
  .metrics-grid {
    grid-template-columns: 1fr;
  }

  .detail-title {
    font-size: 24px;
  }

  .panel-controls {
    justify-content: space-between;
  }

  .control-btn {
    flex: 1;
    min-width: 80px;
  }
}
</style>
