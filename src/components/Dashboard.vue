<script setup>
import { ref, computed, reactive } from 'vue'

const props = defineProps({
  user: String
})

const emit = defineEmits(['logout'])

const currentView = ref('main') // main, detail, logs, thresholds
const activeTab = ref('monitoring')
const selectedSystem = ref(null)
const soundEnabled = ref(true)

const acbStatus = ref({
  'panel1-a': 'CLOSED',
  'panel1-b': 'CLOSED',
  'panel2-a': 'CLOSED',
  'panel2-b': 'CLOSED'
})

const systems = ref({
  dse1: {
    id: 'dse1',
    name: 'DSE 1',
    description: '1 Panel Daya (ABB M1M) • Suhu Ruangan',
    icon: '⚙️',
    fullDesc: '1 Panel Daya (ABB M1M) - DSE 1 Monitoring & Kontrol'
  },
  dse2: {
    id: 'dse2',
    name: 'DSE 2',
    description: '1 Panel Daya (ABB M1M) • Suhu Ruangan',
    icon: '⚙️',
    fullDesc: '1 Panel Daya (ABB M1M) - DSE 2 Monitoring & Kontrol'
  },
  panel1: {
    id: 'panel1',
    name: 'Panel Powermeter 1',
    description: 'Power Meter & Kontrol ACB • Suhu Ruangan',
    icon: '📊',
    fullDesc: 'Panel Powermeter 1 dengan Kontrol ACB'
  },
  panel2: {
    id: 'panel2',
    name: 'Panel Powermeter 2',
    description: 'Power Meter & Kontrol ACB • Suhu Ruangan',
    icon: '📊',
    fullDesc: 'Panel Powermeter 2 dengan Kontrol ACB'
  }
})

const logsData = ref([
  { time: '11/8/2026, 16:02.45', room: 'Panel 1', status: 'ALARM', detail: 'Voltage over high (235V > 242V threshold)' },
  { time: '11/8/2026, 16:01.32', room: 'DSE 2', status: 'Normal', detail: 'temp: 29.1°C, voltage: 232V, current: 45A' },
  { time: '11/8/2026, 16:00.15', room: 'Panel 2', status: 'ALARM', detail: 'Current over high (52A > 50A threshold)' },
  { time: '11/8/2026, 15:59.52', room: 'DSE 1', status: 'Normal', detail: 'temp: 28.5°C, voltage: 231V, current: 42A' },
  { time: '11/8/2026, 15:58.30', room: 'Panel 1', status: 'ALARM', detail: 'Frequency above high (50.65Hz > 50.5Hz)' },
  { time: '11/8/2026, 15:57.18', room: 'DSE 2', status: 'ALARM', detail: 'Over frequency (50.6Hz > 50.5Hz)' },
  { time: '11/8/2026, 15:56.05', room: 'Panel 2', status: 'Normal', detail: 'temp: 28.9°C, voltage: 233V, current: 48A' },
  { time: '11/8/2026, 15:54.40', room: 'DSE 1', status: 'ALARM', detail: 'Temperature high (31.2°C > 30°C threshold)' },
  { time: '11/8/2026, 15:53.22', room: 'Panel 1', status: 'Normal', detail: 'temp: 28.5°C, voltage: 231V, power: 22.5kW' },
  { time: '11/8/2026, 15:52.10', room: 'DSE 2', status: 'Normal', detail: 'All systems normal' },
  { time: '11/8/2026, 15:49.16', room: 'DSE 1', status: 'Normal', detail: 'System initialized' },
])

const logFilter = ref('all')
const filteredLogs = computed(() => {
  if (logFilter.value === 'alarms') {
    return logsData.value.filter(l => l.status === 'ALARM')
  }
  if (logFilter.value === 'normal') {
    return logsData.value.filter(l => l.status === 'Normal')
  }
  return logsData.value
})

const logStats = computed(() => ({
  total: logsData.value.length,
  alarms: logsData.value.filter(l => l.status === 'ALARM').length,
  normal: logsData.value.filter(l => l.status === 'Normal').length,
  rooms: new Set(logsData.value.map(l => l.room)).size
}))

const thresholds = reactive({
  voltage: { base: 220, low: 198, high: 242 },
  current: { base: 42, low: 0, high: 63 },
  frequency: { base: 50, low: 49.5, high: 50.5 },
  power: { base: 18.4, low: 0, high: 35 },
  temp: { base: 27, low: 15, high: 35 },
  rpm: { base: 1500, low: 1450, high: 1550 },
})

const selectSystem = (systemId) => {
  selectedSystem.value = systemId
  currentView.value = 'detail'
  activeTab.value = 'monitoring'
}

const backToMain = () => {
  currentView.value = 'main'
  selectedSystem.value = null
}

const isAdmin = computed(() => props.user === 'admin')

// Sample real-time data with some exceeding thresholds
const sampleData = ref({
  'panel1-a': { voltage: 245, current: 55, frequency: 50.8, temp: 32, power: 25 }, // ALARM: Voltage HIGH
  'panel1-b': { voltage: 190, current: 42, frequency: 49.2, temp: 28, power: 22 }, // ALARM: Voltage LOW, Frequency LOW
  'panel2-a': { voltage: 220, current: 42, frequency: 50.5, temp: 27, power: 20 }, // NORMAL
  'panel2-b': { voltage: 235, current: 48, frequency: 50.6, temp: 31, power: 24 }  // ALARM: Frequency HIGH, Temp HIGH
})

// Check if metrics exceed thresholds and trigger alarm
const checkThresholds = () => {
  let alarmTriggered = false
  let alarmMessages = []

  Object.entries(sampleData.value).forEach(([panelId, data]) => {
    if (data.voltage > thresholds.voltage.high) {
      alarmTriggered = true
      alarmMessages.push(`${panelId}: Voltage HIGH (${data.voltage}V > ${thresholds.voltage.high}V)`)
    }
    if (data.voltage < thresholds.voltage.low) {
      alarmTriggered = true
      alarmMessages.push(`${panelId}: Voltage LOW (${data.voltage}V < ${thresholds.voltage.low}V)`)
    }
    if (data.current > thresholds.current.high) {
      alarmTriggered = true
      alarmMessages.push(`${panelId}: Current HIGH (${data.current}A > ${thresholds.current.high}A)`)
    }
    if (data.frequency > thresholds.frequency.high) {
      alarmTriggered = true
      alarmMessages.push(`${panelId}: Frequency HIGH (${data.frequency}Hz > ${thresholds.frequency.high}Hz)`)
    }
    if (data.frequency < thresholds.frequency.low) {
      alarmTriggered = true
      alarmMessages.push(`${panelId}: Frequency LOW (${data.frequency}Hz < ${thresholds.frequency.low}Hz)`)
    }
    if (data.temp > thresholds.temp.high) {
      alarmTriggered = true
      alarmMessages.push(`${panelId}: Temperature HIGH (${data.temp}°C > ${thresholds.temp.high}°C)`)
    }
    if (data.temp < thresholds.temp.low) {
      alarmTriggered = true
      alarmMessages.push(`${panelId}: Temperature LOW (${data.temp}°C < ${thresholds.temp.low}°C)`)
    }
  })

  if (alarmTriggered) {
    playAlarmSound()
    alert(`🚨 ALARM TRIGGERED!\n\n${alarmMessages.join('\n')}`)
  } else {
    alert('✅ All systems NORMAL - No alarms')
  }
}

const controlACB = (action, panelId) => {
  if (!isAdmin.value) {
    alert('Only Admin can control ACB!')
    return
  }

  const actions = {
    'open': { text: 'ACB Dibuka', status: 'OPEN' },
    'trip': { text: 'ACB di-Trip', status: 'TRIP' },
    'close': { text: 'ACB Ditutup', status: 'CLOSED' }
  }

  // Update ACB status
  acbStatus.value[panelId] = actions[action].status

  // Show notification
  alert(`${actions[action].text} - Success!`)
}

const goToLogs = () => {
  currentView.value = 'logs'
}

const goToThresholds = () => {
  currentView.value = 'thresholds'
}

const exportCSV = () => {
  alert('Exporting to CSV...')
}

const exportExcel = () => {
  alert('Exporting to Excel...')
}

const clearLogs = () => {
  if (confirm('Clear all logs?')) {
    logsData.value = []
  }
}

const saveThresholds = () => {
  alert('Thresholds saved!')
}

const resetThresholds = () => {
  if (confirm('Reset to defaults?')) {
    thresholds.voltage = { base: 220, low: 198, high: 242 }
    thresholds.current = { base: 42, low: 0, high: 63 }
    thresholds.frequency = { base: 50, low: 49.5, high: 50.5 }
    thresholds.power = { base: 18.4, low: 0, high: 35 }
    thresholds.temp = { base: 27, low: 15, high: 35 }
    thresholds.rpm = { base: 1500, low: 1450, high: 1550 }
  }
}

// Alarm sound - plays for 10 seconds only
const playAlarmSound = () => {
  if (!soundEnabled.value) return

  try {
    const audioContext = new (window.AudioContext || window.webkitAudioContext)()
    const now = audioContext.currentTime
    const duration = 10 // 10 seconds only

    // Create oscillator and gain
    const osc = audioContext.createOscillator()
    const gain = audioContext.createGain()

    osc.connect(gain)
    gain.connect(audioContext.destination)

    // 800Hz alarm tone
    osc.frequency.value = 800

    // Start alarm
    osc.start(now)
    gain.gain.setValueAtTime(0.3, now)

    // Fade out at 10 seconds
    gain.gain.setValueAtTime(0.3, now + duration - 0.5)
    gain.gain.linearRampToValueAtTime(0, now + duration)

    // Stop at exactly 10 seconds
    osc.stop(now + duration)
  } catch (e) {
    console.log('Audio not supported')
  }
}

const toggleSound = () => {
  soundEnabled.value = !soundEnabled.value
}

// Simple test function
const testClick = () => {
  alert('Button works!')
  console.log('Button clicked!')
}
</script>

<template>
  <div class="dashboard">
    <!-- Header -->
    <div class="header">
      <div class="header-left">
        <div class="header-brand">AIRNAV</div>
        <div class="header-title">Monitoring dan Kontrol Panel Room</div>
        <div class="header-desc">Pilih ruangan atau sistem yang ingin dipantau. Mode monitoring dan kontrol berlaku untuk seluruh sistem.</div>
      </div>
      <div class="header-right">
        <button class="header-btn" @click="toggleSound">
          {{ soundEnabled ? '🔊 Sound enabled' : '🔇 Sound disabled' }}
        </button>
        <button class="header-btn logout" @click="$emit('logout')">Log out</button>
        <div class="user-status">Signed in as <strong>{{ user }}</strong></div>
      </div>
    </div>

    <!-- Main Tabs (only on main and detail views) -->
    <div v-if="currentView === 'main' || currentView === 'detail'" class="tabs">
      <button class="tab" :class="{ active: activeTab === 'monitoring' }" @click="activeTab = 'monitoring'">
        Monitoring Mode
      </button>
      <button class="tab" :class="{ active: activeTab === 'control' }" @click="activeTab = 'control'">
        Control Mode
      </button>
      <div class="tab-spacer"></div>
      <button class="tab-button" @click="goToLogs">Logs</button>
      <button class="tab-button" @click="goToThresholds">Thresholds</button>
    </div>

    <!-- Detail View Header (for logs/thresholds) -->
    <div v-if="currentView === 'logs' || currentView === 'thresholds'" class="detail-header-bar">
      <button class="back-menu" @click="backToMain">← Menu utama</button>
      <div class="spacer"></div>
      <button class="detail-nav-btn" :class="{ active: true }">{{ currentView === 'logs' ? 'Monitoring mode' : 'Monitoring mode' }}</button>
      <div class="user-status-detail">Signed in as <strong>{{ user }}</strong></div>
      <button class="header-btn logout" @click="$emit('logout')">Log out</button>
    </div>

    <!-- Content -->
    <div class="content">
      <!-- Main Menu -->
      <div v-if="currentView === 'main'" class="systems-grid">
        <div v-for="(system, key) in systems" :key="key" class="system-card" @click="selectSystem(key)">
          <div class="card-icon">{{ system.icon }}</div>
          <div class="card-title">{{ system.name }}</div>
          <div class="card-description">{{ system.description }}</div>
        </div>
      </div>

      <!-- Detail View -->
      <div v-if="currentView === 'detail'" class="detail-view">
        <button class="back-button" @click="backToMain">← Menu utama</button>

        <div class="detail-header">
          <h2 class="detail-title">{{ systems[selectedSystem]?.name }}</h2>
          <p class="detail-desc">{{ systems[selectedSystem]?.fullDesc }}</p>
        </div>

        <!-- Temperature Card -->
        <div class="temp-card">
          <div class="temp-icon">🌡️</div>
          <div class="temp-label">Temperature</div>
          <div class="temp-value">29.3°C</div>
          <div class="temp-chart"></div>
        </div>

        <!-- System Panels -->
        <div class="panels-grid">
          <div class="panel">
            <div class="panel-header">
              <div class="panel-title">{{ systems[selectedSystem]?.name }} A</div>
              <div class="panel-actions">
                <button class="btn-logs" @click="checkThresholds">Check Alert</button>
                <button class="btn-open">Open</button>
              </div>
            </div>
            <div class="panel-info">ABB M1M 20</div>
            <div class="metrics-row">
              <div class="metric">
                <div class="metric-icon">⚡</div>
                <div class="metric-label">Voltage</div>
                <div class="metric-value">230V</div>
                <div class="metric-line"></div>
              </div>
              <div class="metric">
                <div class="metric-icon">⚡</div>
                <div class="metric-label">Current</div>
                <div class="metric-value">45A</div>
                <div class="metric-line"></div>
              </div>
              <div class="metric">
                <div class="metric-icon">◿</div>
                <div class="metric-label">Frequency</div>
                <div class="metric-value">50.4Hz</div>
                <div class="metric-line"></div>
              </div>
            </div>
            <div class="metrics-row">
              <div class="metric">
                <div class="metric-icon">🔌</div>
                <div class="metric-label">Power Factor</div>
                <div class="metric-value">0.87</div>
                <div class="metric-line"></div>
              </div>
              <div class="metric">
                <div class="metric-icon">⚡</div>
                <div class="metric-label">Power</div>
                <div class="metric-value">23.8kW</div>
                <div class="metric-line"></div>
              </div>
            </div>
            <div class="control-section">
              <div class="control-header">
                <div class="control-title">ACB Control</div>
                <div class="acb-status" :class="`status-${acbStatus[selectedSystem + '-a']?.toLowerCase()}`">
                  {{ acbStatus[selectedSystem + '-a'] }}
                </div>
              </div>
              <div v-if="isAdmin" class="control-buttons">
                <button type="button" class="btn-control btn-control-open" @click="controlACB('open', selectedSystem + '-a')">Open</button>
                <button type="button" class="btn-control btn-control-trip" @click="controlACB('trip', selectedSystem + '-a')">Trip</button>
                <button type="button" class="btn-control btn-control-close" @click="controlACB('close', selectedSystem + '-a')">Close</button>
              </div>
              <div v-else class="permission-notice">
                ⚠️ Only Admin can control ACB
              </div>
            </div>
          </div>

          <div class="panel">
            <div class="panel-header">
              <div class="panel-title">{{ systems[selectedSystem]?.name }} B</div>
              <div class="panel-actions">
                <button class="btn-logs" @click="checkThresholds">Check Alert</button>
                <button class="btn-open">Open</button>
              </div>
            </div>
            <div class="panel-info">ABB M1M 20</div>
            <div class="metrics-row">
              <div class="metric">
                <div class="metric-icon">⚡</div>
                <div class="metric-label">Voltage</div>
                <div class="metric-value">231V</div>
                <div class="metric-line"></div>
              </div>
              <div class="metric">
                <div class="metric-icon">⚡</div>
                <div class="metric-label">Current</div>
                <div class="metric-value">48A</div>
                <div class="metric-line"></div>
              </div>
              <div class="metric">
                <div class="metric-icon">◿</div>
                <div class="metric-label">Frequency</div>
                <div class="metric-value">50.3Hz</div>
                <div class="metric-line"></div>
              </div>
            </div>
            <div class="metrics-row">
              <div class="metric">
                <div class="metric-icon">🔌</div>
                <div class="metric-label">Power Factor</div>
                <div class="metric-value">0.90</div>
                <div class="metric-line"></div>
              </div>
              <div class="metric">
                <div class="metric-icon">⚡</div>
                <div class="metric-label">Power</div>
                <div class="metric-value">25.3kW</div>
                <div class="metric-line"></div>
              </div>
            </div>
            <div class="control-section">
              <div class="control-header">
                <div class="control-title">ACB Control</div>
                <div class="acb-status" :class="`status-${acbStatus[selectedSystem + '-b']?.toLowerCase()}`">
                  {{ acbStatus[selectedSystem + '-b'] }}
                </div>
              </div>
              <div v-if="isAdmin" class="control-buttons">
                <button class="btn-control btn-control-open" @click="controlACB('open', selectedSystem + '-b')">Open</button>
                <button class="btn-control btn-control-trip" @click="controlACB('trip', selectedSystem + '-b')">Trip</button>
                <button class="btn-control btn-control-close" @click="controlACB('close', selectedSystem + '-b')">Close</button>
              </div>
              <div v-else class="permission-notice">
                ⚠️ Only Admin can control ACB
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Logs View -->
      <div v-if="currentView === 'logs'" class="logs-view">
        <h2 class="page-title">Log Terpusat</h2>
        <p class="page-desc">Riwayat gabungan seluruh ruangan — bisa difilter, diekspor, dan dibersihkan.</p>

        <div class="logs-header">
          <div class="stats-row">
            <div class="stat">
              <div class="stat-label">TOTAL ENTRI</div>
              <div class="stat-value">{{ logStats.total }}</div>
            </div>
            <div class="stat">
              <div class="stat-label">ALARM</div>
              <div class="stat-value alarm">{{ logStats.alarms }}</div>
            </div>
            <div class="stat">
              <div class="stat-label">NORMAL</div>
              <div class="stat-value">{{ logStats.normal }}</div>
            </div>
            <div class="stat">
              <div class="stat-label">RUANGAN TERDAMPAK</div>
              <div class="stat-value">{{ logStats.rooms }}</div>
            </div>
          </div>

          <div class="logs-actions">
            <button class="btn-export" @click="exportCSV">Export CSV</button>
            <button class="btn-export" @click="exportExcel">Export Excel</button>
            <button class="btn-clear" @click="clearLogs">Clear log</button>
          </div>
        </div>

        <div class="filters">
          <button class="filter-btn" :class="{ active: logFilter === 'all' }" @click="logFilter = 'all'">All</button>
          <button class="filter-btn" :class="{ active: logFilter === 'alarms' }" @click="logFilter = 'alarms'">Alarms</button>
          <button class="filter-btn" :class="{ active: logFilter === 'normal' }" @click="logFilter = 'normal'">Normal</button>
          <select class="filter-select">
            <option>Semua ruangan</option>
            <option>DSE 1</option>
            <option>DSE 2</option>
            <option>Panel 1</option>
            <option>Panel 2</option>
          </select>
        </div>

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
            <tr v-for="(log, idx) in filteredLogs" :key="idx">
              <td>{{ log.time }}</td>
              <td>{{ log.room }}</td>
              <td><span class="badge" :class="log.status === 'Normal' ? 'badge-normal' : 'badge-alarm'">{{ log.status }}</span></td>
              <td>{{ log.detail }}</td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- Thresholds View -->
      <div v-if="currentView === 'thresholds'" class="thresholds-view">
        <div class="threshold-box">
          <h2 class="threshold-title">Threshold editor</h2>
          <p class="threshold-desc">Edit alarm thresholds for metrics. Changes are stored locally.</p>

          <div class="threshold-actions">
            <button class="btn-reset" @click="resetThresholds">Reset</button>
            <button class="btn-save" @click="saveThresholds">Save</button>
          </div>

          <div class="threshold-grid">
            <div v-for="(values, key) in thresholds" :key="key" class="threshold-item">
              <div class="threshold-label">{{ key.charAt(0).toUpperCase() + key.slice(1) }}</div>
              <div class="threshold-fields">
                <div class="field">
                  <label>Base</label>
                  <input v-model.number="values.base" type="number" step="0.1">
                </div>
                <div class="field">
                  <label>Low</label>
                  <input v-model.number="values.low" type="number" step="0.1">
                </div>
                <div class="field">
                  <label>High</label>
                  <input v-model.number="values.high" type="number" step="0.1">
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.dashboard {
  width: 100%;
  min-height: 100vh;
  background: #0f1419;
  display: flex;
  flex-direction: column;
}

/* Header */
.header {
  background: linear-gradient(135deg, #0f1419 0%, #1a2332 100%);
  border-bottom: 1px solid rgba(76, 219, 189, 0.2);
  padding: 20px 40px;
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 40px;
}

.header-left {
  flex: 1;
}

.header-brand {
  font-size: 11px;
  letter-spacing: 2px;
  color: #4cdbbd;
  text-transform: uppercase;
  margin-bottom: 8px;
  font-weight: 600;
}

.header-title {
  font-size: 28px;
  font-weight: 600;
  color: #ffffff;
  margin-bottom: 8px;
}

.header-desc {
  font-size: 13px;
  color: #a0aec0;
  line-height: 1.5;
}

.header-right {
  display: flex;
  align-items: center;
  gap: 12px;
  flex-direction: column;
  align-items: flex-end;
}

.header-btn {
  padding: 8px 16px;
  background: transparent;
  border: none;
  color: #e0e0e0;
  font-size: 13px;
  cursor: pointer;
  transition: color 0.3s ease;
  pointer-events: auto;
  z-index: 20;
}

.header-btn:active {
  transform: scale(0.95);
}

.header-btn:hover {
  color: #4cdbbd;
}

.header-btn.logout {
  color: #4cdbbd;
}

.user-status {
  font-size: 12px;
  color: #a0aec0;
}

.user-status strong {
  color: #ffffff;
}

.user-status-detail {
  font-size: 12px;
  color: #a0aec0;
}

.user-status-detail strong {
  color: #ffffff;
}

/* Tabs */
.tabs {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 20px 40px;
  background: #0f1419;
  border-bottom: 1px solid rgba(76, 219, 189, 0.2);
}

.tab {
  padding: 10px 24px;
  background: rgba(76, 219, 189, 0.1);
  border: none;
  border-radius: 20px;
  color: #a0aec0;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  transition: all 0.3s ease;
}

.tab.active {
  background: #4cdbbd;
  color: #0f1419;
}

.tab:hover {
  background: rgba(76, 219, 189, 0.2);
}

.tab-spacer {
  flex: 1;
}

.tab-button {
  padding: 8px 20px;
  background: transparent;
  border: 1px solid rgba(76, 219, 189, 0.3);
  border-radius: 6px;
  color: #a0aec0;
  cursor: pointer;
  font-size: 13px;
  font-weight: 500;
  transition: all 0.3s ease;
}

.tab-button:hover {
  border-color: #4cdbbd;
  color: #4cdbbd;
}

/* Detail Header Bar (for logs/thresholds) */
.detail-header-bar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 12px 40px;
  background: #0f1419;
  border-bottom: 1px solid rgba(76, 219, 189, 0.2);
  gap: 20px;
}

.back-menu {
  padding: 8px 16px;
  background: rgba(76, 219, 189, 0.1);
  border: 1px solid rgba(76, 219, 189, 0.3);
  border-radius: 6px;
  color: #4cdbbd;
  cursor: pointer;
  font-size: 13px;
  font-weight: 500;
  transition: all 0.3s ease;
}

.back-menu:hover {
  background: rgba(76, 219, 189, 0.2);
}

.spacer {
  flex: 1;
}

.detail-nav-btn {
  padding: 8px 20px;
  background: rgba(76, 219, 189, 0.1);
  border: 1px solid rgba(76, 219, 189, 0.3);
  border-radius: 6px;
  color: #4cdbbd;
  cursor: pointer;
  font-size: 13px;
  font-weight: 500;
}

.detail-nav-btn.active {
  background: rgba(76, 219, 189, 0.2);
}

/* Content */
.content {
  flex: 1;
  padding: 40px;
  overflow-y: auto;
}

/* Systems Grid */
.systems-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 25px;
}

.system-card {
  background: linear-gradient(135deg, rgba(26, 35, 50, 0.8) 0%, rgba(15, 20, 25, 0.8) 100%);
  border: 1px solid rgba(76, 219, 189, 0.2);
  border-radius: 16px;
  padding: 24px;
  cursor: pointer;
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
}

.system-card::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 2px;
  background: linear-gradient(90deg, transparent, #4cdbbd, transparent);
}

.system-card:hover {
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

.card-description {
  font-size: 12px;
  color: #a0aec0;
}

/* Detail View */
.detail-view {
  max-width: 1400px;
}

.back-button {
  padding: 8px 16px;
  background: rgba(76, 219, 189, 0.1);
  border: 1px solid rgba(76, 219, 189, 0.3);
  border-radius: 6px;
  color: #4cdbbd;
  cursor: pointer;
  font-size: 13px;
  margin-bottom: 30px;
  transition: all 0.3s ease;
}

.back-button:hover {
  background: rgba(76, 219, 189, 0.2);
}

.detail-header {
  margin-bottom: 30px;
}

.detail-title {
  font-size: 28px;
  font-weight: 600;
  color: #ffffff;
  margin-bottom: 8px;
}

.detail-desc {
  font-size: 13px;
  color: #a0aec0;
}

/* Temperature Card */
.temp-card {
  background: linear-gradient(135deg, rgba(26, 35, 50, 0.8) 0%, rgba(15, 20, 25, 0.8) 100%);
  border: 1px solid rgba(76, 219, 189, 0.2);
  border-radius: 12px;
  padding: 20px;
  margin-bottom: 30px;
  max-width: 280px;
}

.temp-icon {
  font-size: 20px;
  margin-bottom: 8px;
}

.temp-label {
  font-size: 12px;
  color: #a0aec0;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: 8px;
}

.temp-value {
  font-size: 24px;
  font-weight: 600;
  color: #4cdbbd;
  margin-bottom: 8px;
}

.temp-chart {
  height: 40px;
  background: linear-gradient(90deg, #4cdbbd, rgba(76, 219, 189, 0.2));
  border-radius: 4px;
}

/* Panels Grid */
.panels-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(350px, 1fr));
  gap: 25px;
  margin-bottom: 30px;
}

.panel {
  background: linear-gradient(135deg, rgba(26, 35, 50, 0.8) 0%, rgba(15, 20, 25, 0.8) 100%);
  border: 1px solid rgba(76, 219, 189, 0.2);
  border-radius: 16px;
  padding: 24px;
}

.panel-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.panel-title {
  font-size: 16px;
  font-weight: 600;
  color: #ffffff;
}

.panel-actions {
  display: flex;
  gap: 8px;
}

.btn-logs {
  padding: 8px 14px;
  background: rgba(76, 219, 189, 0.15);
  border: 1px solid rgba(76, 219, 189, 0.4);
  border-radius: 6px;
  color: #4cdbbd;
  cursor: pointer;
  font-size: 12px;
  font-weight: 600;
  transition: all 0.2s ease;
  pointer-events: auto;
  z-index: 10;
}

.btn-logs:hover {
  background: rgba(76, 219, 189, 0.3);
  border-color: #4cdbbd;
  box-shadow: 0 2px 8px rgba(76, 219, 189, 0.2);
}

.btn-logs:active {
  transform: scale(0.95);
}

.btn-open {
  padding: 8px 14px;
  background: rgba(76, 219, 189, 0.2);
  border: 1px solid #4cdbbd;
  border-radius: 6px;
  color: #4cdbbd;
  cursor: pointer;
  font-size: 12px;
  font-weight: 600;
  transition: all 0.2s ease;
  pointer-events: auto;
  z-index: 10;
}

.btn-open:hover {
  background: rgba(76, 219, 189, 0.4);
  box-shadow: 0 2px 8px rgba(76, 219, 189, 0.3);
}

.btn-open:active {
  transform: scale(0.95);
}

.panel-info {
  font-size: 12px;
  color: #a0aec0;
  margin-bottom: 16px;
}

.metrics-row {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 12px;
  margin-bottom: 16px;
}

.metrics-row:last-child {
  margin-bottom: 0;
}

.metric {
  background: rgba(26, 35, 50, 0.6);
  border: 1px solid rgba(76, 219, 189, 0.15);
  border-radius: 8px;
  padding: 12px;
}

.metric-icon {
  font-size: 16px;
  margin-bottom: 4px;
}

.metric-label {
  font-size: 11px;
  color: #a0aec0;
  text-transform: uppercase;
  letter-spacing: 0.4px;
  margin-bottom: 4px;
}

.metric-value {
  font-size: 18px;
  font-weight: 600;
  color: #4cdbbd;
  margin-bottom: 4px;
}

.metric-line {
  height: 3px;
  background: linear-gradient(90deg, #4cdbbd, rgba(76, 219, 189, 0.2));
  border-radius: 2px;
}

/* Control Section */
.control-section {
  margin-top: 16px;
  padding-top: 16px;
  border-top: 1px solid rgba(76, 219, 189, 0.15);
}

.control-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.control-title {
  font-size: 12px;
  color: #a0aec0;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  font-weight: 600;
}

.acb-status {
  padding: 4px 12px;
  border-radius: 4px;
  font-size: 11px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.4px;
}

.acb-status.status-open {
  background: rgba(34, 197, 94, 0.2);
  color: #22c55e;
}

.acb-status.status-closed {
  background: rgba(76, 219, 189, 0.2);
  color: #4cdbbd;
}

.acb-status.status-trip {
  background: rgba(239, 68, 68, 0.2);
  color: #ef4444;
}

.control-buttons {
  display: flex;
  gap: 8px;
}

.btn-control {
  flex: 1;
  padding: 10px 12px;
  border: none;
  border-radius: 6px;
  font-size: 12px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  pointer-events: auto;
  z-index: 10;
}

.btn-control:active {
  transform: scale(0.95);
}

.btn-control-open {
  background: rgba(76, 219, 189, 0.2);
  color: #4cdbbd;
  border: 1px solid #4cdbbd;
}

.btn-control-open:hover {
  background: rgba(76, 219, 189, 0.3);
  box-shadow: 0 4px 12px rgba(76, 219, 189, 0.2);
}

.btn-control-trip {
  background: rgba(249, 115, 22, 0.2);
  color: #f97316;
  border: 1px solid #f97316;
}

.btn-control-trip:hover {
  background: rgba(249, 115, 22, 0.3);
  box-shadow: 0 4px 12px rgba(249, 115, 22, 0.2);
}

.btn-control-close {
  background: rgba(239, 68, 68, 0.2);
  color: #ef4444;
  border: 1px solid #ef4444;
}

.btn-control-close:hover {
  background: rgba(239, 68, 68, 0.3);
  box-shadow: 0 4px 12px rgba(239, 68, 68, 0.2);
}

.permission-notice {
  padding: 10px 12px;
  background: rgba(249, 115, 22, 0.1);
  border: 1px solid rgba(249, 115, 22, 0.3);
  border-radius: 6px;
  color: #f97316;
  font-size: 12px;
  font-weight: 600;
  text-align: center;
}

/* Logs View */
.logs-view {
  max-width: 1400px;
}

.page-title {
  font-size: 28px;
  font-weight: 600;
  color: #ffffff;
  margin-bottom: 8px;
}

.page-desc {
  font-size: 13px;
  color: #a0aec0;
  margin-bottom: 30px;
}

.logs-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 20px;
  margin-bottom: 25px;
  flex-wrap: wrap;
}

.stats-row {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 15px;
  flex: 1;
  min-width: 500px;
}

.stat {
  background: rgba(26, 35, 50, 0.6);
  border: 1px solid rgba(76, 219, 189, 0.2);
  border-radius: 8px;
  padding: 20px;
  text-align: center;
}

.stat-label {
  font-size: 11px;
  color: #a0aec0;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: 10px;
  font-weight: 600;
}

.stat-value {
  font-size: 28px;
  font-weight: 600;
  color: #4cdbbd;
}

.stat-value.alarm {
  color: #ef4444;
}

.logs-actions {
  display: flex;
  gap: 10px;
  flex-direction: column;
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
  white-space: nowrap;
}

.btn-export:hover {
  background: rgba(76, 219, 189, 0.3);
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
  white-space: nowrap;
}

.btn-clear:hover {
  background: rgba(239, 68, 68, 0.3);
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

.filter-select {
  padding: 8px 16px;
  background: rgba(26, 35, 50, 0.6);
  border: 1px solid rgba(76, 219, 189, 0.3);
  border-radius: 6px;
  color: #a0aec0;
  cursor: pointer;
  font-size: 13px;
}

.logs-table {
  width: 100%;
  border-collapse: collapse;
  background: linear-gradient(135deg, rgba(26, 35, 50, 0.6) 0%, rgba(15, 20, 25, 0.6) 100%);
  border: 1px solid rgba(76, 219, 189, 0.2);
  border-radius: 8px;
  overflow: hidden;
}

.logs-table thead {
  border-bottom: 1px solid rgba(76, 219, 189, 0.2);
}

.logs-table th {
  padding: 16px;
  text-align: left;
  font-size: 11px;
  font-weight: 600;
  color: #4cdbbd;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  background: rgba(76, 219, 189, 0.05);
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

.badge {
  display: inline-block;
  padding: 4px 12px;
  border-radius: 12px;
  font-size: 11px;
  font-weight: 600;
}

.badge-normal {
  background: rgba(76, 219, 189, 0.2);
  color: #4cdbbd;
}

.badge-alarm {
  background: rgba(239, 68, 68, 0.2);
  color: #ef4444;
}

/* Thresholds View */
.thresholds-view {
  max-width: 1400px;
}

.threshold-box {
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

.threshold-desc {
  font-size: 13px;
  color: #a0aec0;
  margin-bottom: 25px;
}

.threshold-actions {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  margin-bottom: 25px;
}

.btn-reset {
  padding: 10px 20px;
  background: rgba(76, 219, 189, 0.1);
  border: 1px solid rgba(76, 219, 189, 0.5);
  border-radius: 6px;
  color: #4cdbbd;
  cursor: pointer;
  font-size: 13px;
  font-weight: 600;
  transition: all 0.3s ease;
}

.btn-reset:hover {
  background: rgba(76, 219, 189, 0.2);
}

.btn-save {
  padding: 10px 20px;
  background: #4cdbbd;
  border: none;
  border-radius: 6px;
  color: #0f1419;
  cursor: pointer;
  font-size: 13px;
  font-weight: 600;
  transition: all 0.3s ease;
}

.btn-save:hover {
  background: #3ac9ad;
  box-shadow: 0 5px 15px rgba(76, 219, 189, 0.3);
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

.threshold-label {
  font-size: 13px;
  color: #a0aec0;
  margin-bottom: 12px;
  font-weight: 600;
  text-transform: capitalize;
}

.threshold-fields {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.field {
  display: flex;
  flex-direction: column;
}

.field label {
  font-size: 10px;
  color: #718096;
  text-transform: uppercase;
  letter-spacing: 0.4px;
  margin-bottom: 4px;
  font-weight: 600;
}

.field input {
  padding: 8px 12px;
  background: rgba(15, 20, 25, 0.6);
  border: 1px solid rgba(76, 219, 189, 0.2);
  border-radius: 4px;
  color: #ffffff;
  font-size: 13px;
  transition: all 0.3s ease;
}

.field input:focus {
  outline: none;
  border-color: #4cdbbd;
  box-shadow: 0 0 0 2px rgba(76, 219, 189, 0.1);
}

@media (max-width: 1024px) {
  .header {
    flex-direction: column;
    gap: 20px;
  }

  .stats-row {
    grid-template-columns: repeat(2, 1fr);
    min-width: auto;
  }

  .panels-grid {
    grid-template-columns: 1fr;
  }

  .metrics-row {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 768px) {
  .header {
    padding: 20px;
  }

  .content {
    padding: 20px;
  }

  .tabs {
    padding: 20px;
  }

  .systems-grid {
    grid-template-columns: 1fr;
  }

  .stats-row {
    grid-template-columns: repeat(2, 1fr);
  }

  .metrics-row {
    grid-template-columns: 1fr;
  }

  .logs-header {
    flex-direction: column;
  }

  .logs-actions {
    flex-direction: row;
  }

  .threshold-grid {
    grid-template-columns: 1fr;
  }
}
</style>
