<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  logs: Array
})

const emit = defineEmits(['add-log'])

const filterType = ref('all')

const filteredLogs = computed(() => {
  if (filterType.value === 'alarms') {
    return props.logs.filter(l => l.status === 'ALARM')
  }
  if (filterType.value === 'normal') {
    return props.logs.filter(l => l.status === 'Normal')
  }
  return props.logs
})

const stats = computed(() => ({
  total: props.logs.length,
  alarms: props.logs.filter(l => l.status === 'ALARM').length,
  normal: props.logs.filter(l => l.status === 'Normal').length,
  rooms: new Set(props.logs.map(l => l.room)).size
}))

const exportCSV = () => {
  const csv = [
    ['Waktu', 'Ruangan', 'Status', 'Detail'],
    ...props.logs.map(l => [l.time, l.room, l.status, l.detail])
  ].map(row => row.map(cell => `"${cell}"`).join(',')).join('\n')

  const blob = new Blob([csv], { type: 'text/csv' })
  const url = window.URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = 'logs.csv'
  a.click()
}

const exportExcel = () => {
  alert('Export Excel functionality would require additional library like xlsx')
}

const clearLogs = () => {
  if (confirm('Clear all logs? This action cannot be undone.')) {
    props.logs.length = 0
  }
}

const setFilter = (type) => {
  filterType.value = type
}
</script>

<template>
  <div>
    <div class="logs-header">
      <div>
        <h2 class="logs-title">Log Terpusat</h2>
        <p class="page-description">Riwayat gabungan seluruh ruangan — bisa difilter, diekspor, dan dibersihkan.</p>
      </div>
      <div class="export-buttons">
        <button class="btn-export" @click="exportCSV">Export CSV</button>
        <button class="btn-export" @click="exportExcel">Export Excel</button>
        <button class="btn-clear" @click="clearLogs">Clear log</button>
      </div>
    </div>

    <div class="stats-grid">
      <div class="stat-box">
        <div class="stat-label">Total Entri</div>
        <div class="stat-value">{{ stats.total }}</div>
      </div>
      <div class="stat-box">
        <div class="stat-label">Alarm</div>
        <div class="stat-value alarm">{{ stats.alarms }}</div>
      </div>
      <div class="stat-box">
        <div class="stat-label">Normal</div>
        <div class="stat-value">{{ stats.normal }}</div>
      </div>
      <div class="stat-box">
        <div class="stat-label">Ruangan Terdampak</div>
        <div class="stat-value">{{ stats.rooms }}</div>
      </div>
    </div>

    <div class="filters">
      <button
        class="filter-btn"
        :class="{ active: filterType === 'all' }"
        @click="setFilter('all')"
      >
        All
      </button>
      <button
        class="filter-btn"
        :class="{ active: filterType === 'alarms' }"
        @click="setFilter('alarms')"
      >
        Alarms
      </button>
      <button
        class="filter-btn"
        :class="{ active: filterType === 'normal' }"
        @click="setFilter('normal')"
      >
        Normal
      </button>
      <select class="filter-select">
        <option>Semua ruangan</option>
        <option>DSE 1</option>
        <option>DSE 2</option>
        <option>Panel 1</option>
        <option>Panel 2</option>
      </select>
    </div>

    <div class="logs-table-container">
      <table class="logs-table">
        <thead>
          <tr>
            <th>Waktu</th>
            <th>Ruangan</th>
            <th>Status</th>
            <th>Detail</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(log, index) in filteredLogs" :key="index">
            <td>{{ log.time }}</td>
            <td>{{ log.room }}</td>
            <td>
              <span class="status-badge" :class="log.status === 'Normal' ? 'status-normal' : 'status-alarm'">
                {{ log.status }}
              </span>
            </td>
            <td>{{ log.detail }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<style scoped>
.logs-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 25px;
  gap: 20px;
  flex-wrap: wrap;
}

.logs-title {
  font-size: 28px;
  font-weight: 600;
  color: #ffffff;
  margin-bottom: 8px;
}

.page-description {
  color: #a0aec0;
  font-size: 14px;
}

.export-buttons {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
}

.btn-export {
  padding: 10px 20px;
  background: rgba(76, 219, 189, 0.2);
  border: 1px solid #4cdbbd;
  border-radius: 6px;
  color: #4cdbbd;
  cursor: pointer;
  font-size: 13px;
  font-weight: 600;
  transition: all 0.3s ease;
}

.btn-export:hover {
  background: rgba(76, 219, 189, 0.3);
  box-shadow: 0 5px 15px rgba(76, 219, 189, 0.2);
}

.btn-clear {
  padding: 10px 20px;
  background: rgba(239, 68, 68, 0.2);
  border: 1px solid #ef4444;
  border-radius: 6px;
  color: #ef4444;
  cursor: pointer;
  font-size: 13px;
  font-weight: 600;
  transition: all 0.3s ease;
}

.btn-clear:hover {
  background: rgba(239, 68, 68, 0.3);
  box-shadow: 0 5px 15px rgba(239, 68, 68, 0.2);
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 15px;
  margin-bottom: 25px;
}

.stat-box {
  background: rgba(26, 35, 50, 0.6);
  border: 1px solid rgba(76, 219, 189, 0.2);
  border-radius: 8px;
  padding: 20px;
  text-align: center;
}

.stat-label {
  font-size: 12px;
  color: #a0aec0;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: 10px;
}

.stat-value {
  font-size: 28px;
  font-weight: 600;
  color: #4cdbbd;
}

.stat-value.alarm {
  color: #ef4444;
}

.filters {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
  flex-wrap: wrap;
}

.filter-btn {
  padding: 8px 16px;
  background: rgba(76, 219, 189, 0.1);
  border: 1px solid rgba(76, 219, 189, 0.3);
  border-radius: 6px;
  color: #a0aec0;
  cursor: pointer;
  font-size: 13px;
  transition: all 0.3s ease;
}

.filter-btn.active {
  background: #4cdbbd;
  color: #0f1419;
  border-color: #4cdbbd;
}

.filter-btn:hover {
  border-color: #4cdbbd;
}

.filter-select {
  padding: 8px 16px;
  background: rgba(26, 35, 50, 0.6);
  border: 1px solid rgba(76, 219, 189, 0.3);
  border-radius: 6px;
  color: #a0aec0;
  cursor: pointer;
  font-size: 13px;
}

.logs-table-container {
  overflow-x: auto;
  border-radius: 8px;
  border: 1px solid rgba(76, 219, 189, 0.2);
}

.logs-table {
  width: 100%;
  border-collapse: collapse;
  background: linear-gradient(135deg, rgba(26, 35, 50, 0.6) 0%, rgba(15, 20, 25, 0.6) 100%);
}

.logs-table thead {
  border-bottom: 1px solid rgba(76, 219, 189, 0.2);
}

.logs-table th {
  padding: 16px;
  text-align: left;
  font-size: 12px;
  font-weight: 600;
  color: #4cdbbd;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  background: rgba(76, 219, 189, 0.05);
  white-space: nowrap;
}

.logs-table td {
  padding: 14px 16px;
  font-size: 13px;
  color: #cbd5e0;
  border-bottom: 1px solid rgba(76, 219, 189, 0.1);
}

.logs-table tbody tr:hover {
  background: rgba(76, 219, 189, 0.05);
}

.status-badge {
  display: inline-block;
  padding: 4px 12px;
  border-radius: 12px;
  font-size: 12px;
  font-weight: 600;
}

.status-normal {
  background: rgba(76, 219, 189, 0.2);
  color: #4cdbbd;
}

.status-alarm {
  background: rgba(239, 68, 68, 0.2);
  color: #ef4444;
}

@media (max-width: 1024px) {
  .logs-header {
    flex-direction: column;
  }

  .logs-table {
    font-size: 12px;
  }

  .logs-table td,
  .logs-table th {
    padding: 10px 8px;
  }
}
</style>
