<script setup>
import { ref, reactive } from 'vue'

const thresholdsList = reactive({
  voltage: { base: 220, low: 198, high: 242 },
  current: { base: 42, low: 0, high: 63 },
  frequency: { base: 50, low: 49.5, high: 50.5 },
  power: { base: 18.4, low: 0, high: 35 },
  temp: { base: 27, low: 15, high: 35 },
  rpm: { base: 1500, low: 1450, high: 1550 },
  hz: { base: 50, low: 49.5, high: 50.5 },
  battery: { base: 26.5, low: 24, high: 28.5 },
  voltageNR: { base: 220, low: 198, high: 242 },
  currentNR: { base: 40, low: 0, high: 63 },
  currentOutR: { base: 40, low: 0, high: 63 },
  voltageInS: { base: 220, low: 198, high: 242 },
  voltageOutS: { base: 220, low: 198, high: 242 }
})

const defaultThresholds = {
  voltage: { base: 220, low: 198, high: 242 },
  current: { base: 42, low: 0, high: 63 },
  frequency: { base: 50, low: 49.5, high: 50.5 },
  power: { base: 18.4, low: 0, high: 35 },
  temp: { base: 27, low: 15, high: 35 },
  rpm: { base: 1500, low: 1450, high: 1550 },
  hz: { base: 50, low: 49.5, high: 50.5 },
  battery: { base: 26.5, low: 24, high: 28.5 },
  voltageNR: { base: 220, low: 198, high: 242 },
  currentNR: { base: 40, low: 0, high: 63 },
  currentOutR: { base: 40, low: 0, high: 63 },
  voltageInS: { base: 220, low: 198, high: 242 },
  voltageOutS: { base: 220, low: 198, high: 242 }
}

const saveThresholds = () => {
  // Save to localStorage
  localStorage.setItem('thresholds', JSON.stringify(thresholdsList))
  alert('Thresholds saved locally!')
}

const resetThresholds = () => {
  if (confirm('Reset all thresholds to default?')) {
    Object.assign(thresholdsList, defaultThresholds)
  }
}

const formatLabel = (key) => {
  return key
    .replace(/([A-Z])/g, ' $1')
    .replace(/^./, str => str.toUpperCase())
    .trim()
}
</script>

<template>
  <div class="threshold-editor">
    <div class="threshold-title">Threshold editor</div>
    <div class="threshold-description">Edit alarm thresholds for metrics. Changes are stored locally.</div>

    <div class="threshold-controls">
      <button class="btn-threshold btn-reset" @click="resetThresholds">Reset</button>
      <button class="btn-threshold btn-save" @click="saveThresholds">Save</button>
    </div>

    <div class="threshold-grid">
      <div v-for="(values, key) in thresholdsList" :key="key" class="threshold-item">
        <div class="threshold-item-label">{{ formatLabel(key) }}</div>
        <div class="threshold-field">
          <label>Base</label>
          <input v-model.number="values.base" type="number" step="0.1">
        </div>
        <div class="threshold-field">
          <label>Low</label>
          <input v-model.number="values.low" type="number" step="0.1">
        </div>
        <div class="threshold-field">
          <label>High</label>
          <input v-model.number="values.high" type="number" step="0.1">
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.threshold-editor {
  background: linear-gradient(135deg, rgba(26, 35, 50, 0.8) 0%, rgba(15, 20, 25, 0.8) 100%);
  border: 1px solid rgba(76, 219, 189, 0.2);
  border-radius: 16px;
  padding: 30px;
}

.threshold-title {
  font-size: 20px;
  font-weight: 600;
  color: #ffffff;
  margin-bottom: 8px;
}

.threshold-description {
  font-size: 13px;
  color: #a0aec0;
  margin-bottom: 25px;
}

.threshold-controls {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  margin-bottom: 25px;
}

.btn-threshold {
  padding: 10px 20px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 13px;
  font-weight: 600;
  transition: all 0.3s ease;
}

.btn-reset {
  background: rgba(76, 219, 189, 0.1);
  border: 1px solid rgba(76, 219, 189, 0.5);
  color: #4cdbbd;
}

.btn-reset:hover {
  background: rgba(76, 219, 189, 0.2);
}

.btn-save {
  background: #4cdbbd;
  border: none;
  color: #0f1419;
}

.btn-save:hover {
  background: #3ac9ad;
  box-shadow: 0 5px 15px rgba(76, 219, 189, 0.3);
  transform: translateY(-2px);
}

.threshold-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 20px;
}

.threshold-item {
  background: rgba(26, 35, 50, 0.6);
  border: 1px solid rgba(76, 219, 189, 0.15);
  border-radius: 8px;
  padding: 16px;
}

.threshold-item-label {
  font-size: 13px;
  color: #a0aec0;
  margin-bottom: 12px;
  font-weight: 500;
  text-transform: capitalize;
}

.threshold-field {
  margin-bottom: 12px;
}

.threshold-field:last-child {
  margin-bottom: 0;
}

.threshold-field label {
  font-size: 11px;
  color: #718096;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  display: block;
  margin-bottom: 4px;
  font-weight: 600;
}

.threshold-field input {
  width: 100%;
  padding: 8px 12px;
  background: rgba(15, 20, 25, 0.6);
  border: 1px solid rgba(76, 219, 189, 0.2);
  border-radius: 4px;
  color: #ffffff;
  font-size: 13px;
  transition: all 0.3s ease;
}

.threshold-field input:focus {
  outline: none;
  border-color: #4cdbbd;
  box-shadow: 0 0 0 2px rgba(76, 219, 189, 0.1);
}

.threshold-field input::placeholder {
  color: #718096;
}

@media (max-width: 768px) {
  .threshold-editor {
    padding: 20px;
  }

  .threshold-grid {
    grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  }

  .threshold-controls {
    flex-direction: column-reverse;
  }

  .btn-threshold {
    width: 100%;
  }
}
</style>
