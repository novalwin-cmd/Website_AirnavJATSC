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
    name: 'Panel Technical Genset',
    description: 'Generator Set Technical • Voltage, Current & Frequency Control',
    icon: '⚙️',
    fullDesc: 'Panel Technical Genset - DSE 1 Real-time Monitoring & ACB Control'
  },
  dse2: {
    id: 'dse2',
    name: 'Panel Prioritas Genset',
    description: 'Priority Generator Set • Load Balancing & Protection',
    icon: '⚙️',
    fullDesc: 'Panel Prioritas Genset - DSE 2 Real-time Monitoring & ACB Control'
  },
  panel1: {
    id: 'panel1',
    name: 'Panel A7',
    description: 'Distribution Panel A7 • Power Meter & Load Control',
    icon: '📊',
    fullDesc: 'Panel A7 - Power Distribution & ACB Control with Real-time Metrics'
  },
  panel2: {
    id: 'panel2',
    name: 'Panel T7',
    description: 'Distribution Panel T7 • Power Meter & Load Control',
    icon: '📊',
    fullDesc: 'Panel T7 - Power Distribution & ACB Control with Real-time Metrics'
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
  'dse1-a': { voltage: 228, current: 42, frequency: 50.2, temp: 28.5, power: 18.5 },
  'dse1-b': { voltage: 230, current: 40, frequency: 50.1, temp: 27.2, power: 17.8 },
  'dse2-a': { voltage: 232, current: 45, frequency: 50.3, temp: 29.1, power: 20.2 },
  'dse2-b': { voltage: 235, current: 48, frequency: 50.6, temp: 31.5, power: 24.0 }, // ALARM: Frequency HIGH, Temp HIGH
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

// Get system status based on thresholds
const getSystemStatus = () => {
  if (!selectedSystem.value) return 'normal'

  const data = sampleData.value[selectedSystem.value + '-a']
  if (!data) return 'normal'

  // Check if any metric exceeds thresholds
  if (data.voltage > thresholds.voltage.high || data.voltage < thresholds.voltage.low) return 'critical'
  if (data.current > thresholds.current.high) return 'critical'
  if (data.temp > thresholds.temp.high) return 'critical'
  if (data.frequency > thresholds.frequency.high || data.frequency < thresholds.frequency.low) return 'warning'

  return 'normal'
}

const getSystemStatusText = () => {
  const status = getSystemStatus()
  if (status === 'critical') return '🔴 CRITICAL'
  if (status === 'warning') return '🟡 WARNING'
  return '🟢 NORMAL'
}
</script>

<template>
  <div class="dashboard-hmi">
    <!-- Header -->
    <div class="header-top">
      <div class="header-left">
        <div class="header-brand">AIRNAV</div>
        <div class="header-title">HMI Power Monitoring System</div>
      </div>
      <div class="header-right">
        <button class="header-btn" @click="toggleSound" title="Toggle alarm sound">
          {{ soundEnabled ? '🔊 Sound enabled' : '🔇 Sound disabled' }}
        </button>
        <div class="user-status">Signed in as <strong>{{ user }}</strong></div>
        <button class="header-btn logout" @click="$emit('logout')">Log out</button>
      </div>
    </div>

    <!-- Main Layout: Sidebar + Content -->
    <div class="hmi-container">
      <!-- Left Sidebar Navigation -->
      <div class="sidebar">
        <div class="sidebar-title">Systems</div>
        <div class="nav-buttons">
          <button
            v-for="(system, key) in systems"
            :key="key"
            class="nav-btn"
            :class="{ active: selectedSystem === key }"
            @click="selectSystem(key)"
          >
            <span class="nav-icon">{{ system.icon }}</span>
            <span class="nav-label">{{ system.name }}</span>
          </button>
        </div>

        <div class="sidebar-divider"></div>

        <div class="sidebar-section">
          <div class="sidebar-label">Menu</div>
          <button
            class="nav-btn"
            :class="{ active: currentView === 'logs' }"
            @click="goToLogs"
          >
            <span class="nav-icon">📋</span>
            <span class="nav-label">Logs</span>
          </button>
          <button
            class="nav-btn"
            :class="{ active: currentView === 'thresholds' }"
            @click="goToThresholds"
          >
            <span class="nav-icon">⚙️</span>
            <span class="nav-label">Thresholds</span>
          </button>
        </div>
      </div>

      <!-- Main Content Area -->
      <div class="main-content">
        <!-- Detail View (Monitoring) -->
        <div v-if="currentView === 'detail' && selectedSystem" class="monitoring-view">
          <div class="view-header">
            <div>
              <h2 class="view-title">{{ systems[selectedSystem]?.name }}</h2>
              <p class="view-desc">{{ systems[selectedSystem]?.fullDesc }}</p>
            </div>
            <div class="mode-indicator" :class="{ active: activeTab === 'control' }">
              {{ activeTab === 'monitoring' ? '📊 MONITORING' : '🎛️ CONTROL' }}
            </div>
          </div>

          <!-- Power Meter Panel (Main Display) -->
          <div class="power-meter-panel">
            <div class="power-meter-header">
              <h3>Power Meter</h3>
              <div class="status-badge" :class="getSystemStatus()">{{ getSystemStatusText() }}</div>
            </div>

            <div class="meter-display">
              <div class="meter-item">
                <div class="meter-label">Power</div>
                <div class="meter-value">{{ sampleData[selectedSystem + '-a']?.power || 0 }}kW</div>
                <div class="meter-icon">⚡</div>
              </div>
              <div class="meter-item">
                <div class="meter-label">Voltage</div>
                <div class="meter-value">{{ sampleData[selectedSystem + '-a']?.voltage || 0 }}V</div>
                <div class="meter-icon">⚙️</div>
              </div>
              <div class="meter-item">
                <div class="meter-label">Current</div>
                <div class="meter-value">{{ sampleData[selectedSystem + '-a']?.current || 0 }}A</div>
                <div class="meter-icon">🔌</div>
              </div>
              <div class="meter-item">
                <div class="meter-label">Frequency</div>
                <div class="meter-value">{{ sampleData[selectedSystem + '-a']?.frequency || 0 }}Hz</div>
                <div class="meter-icon">◿</div>
              </div>
            </div>
          </div>

          <!-- ACB Control Panel -->
          <div class="acb-control-panel">
            <div class="acb-header">
              <h3>ACB Control & Status</h3>
              <button class="btn-check-alert" @click="checkThresholds">⚠️ Check Thresholds</button>
            </div>

            <!-- ACB Channel A -->
            <div class="acb-channel">
              <div class="channel-info">
                <div class="channel-name">Channel A - ABB M1M 20</div>
                <div class="acb-status" :class="`status-${acbStatus[selectedSystem + '-a']?.toLowerCase()}`">
                  {{ acbStatus[selectedSystem + '-a'] }}
                </div>
              </div>

              <div class="channel-metrics">
                <div class="metric-card">
                  <span class="metric-label">Voltage</span>
                  <span class="metric-value">{{ sampleData[selectedSystem + '-a']?.voltage }}V</span>
                </div>
                <div class="metric-card">
                  <span class="metric-label">Current</span>
                  <span class="metric-value">{{ sampleData[selectedSystem + '-a']?.current }}A</span>
                </div>
                <div class="metric-card">
                  <span class="metric-label">Power Factor</span>
                  <span class="metric-value">{{ (Math.random() * 0.3 + 0.75).toFixed(2) }}</span>
                </div>
                <div class="metric-card">
                  <span class="metric-label">Frequency</span>
                  <span class="metric-value">{{ sampleData[selectedSystem + '-a']?.frequency }}Hz</span>
                </div>
              </div>

              <div v-if="activeTab === 'control'" class="control-buttons">
                <button
                  v-if="isAdmin"
                  class="btn-acb btn-acb-open"
                  @click="controlACB('open', selectedSystem + '-a')"
                  :disabled="acbStatus[selectedSystem + '-a'] === 'OPEN'"
                >
                  Open
                </button>
                <button
                  v-if="isAdmin"
                  class="btn-acb btn-acb-trip"
                  @click="controlACB('trip', selectedSystem + '-a')"
                >
                  Trip
                </button>
                <button
                  v-if="isAdmin"
                  class="btn-acb btn-acb-close"
                  @click="controlACB('close', selectedSystem + '-a')"
                  :disabled="acbStatus[selectedSystem + '-a'] === 'CLOSED'"
                >
                  Close
                </button>
                <div v-if="!isAdmin" class="permission-notice">Only Admin can control ACB</div>
              </div>
              <div v-else class="control-disabled">Monitoring Mode - Control Disabled</div>
            </div>

            <!-- ACB Channel B -->
            <div class="acb-channel">
              <div class="channel-info">
                <div class="channel-name">Channel B - ABB M1M 20</div>
                <div class="acb-status" :class="`status-${acbStatus[selectedSystem + '-b']?.toLowerCase()}`">
                  {{ acbStatus[selectedSystem + '-b'] }}
                </div>
              </div>

              <div class="channel-metrics">
                <div class="metric-card">
                  <span class="metric-label">Voltage</span>
                  <span class="metric-value">{{ sampleData[selectedSystem + '-b']?.voltage }}V</span>
                </div>
                <div class="metric-card">
                  <span class="metric-label">Current</span>
                  <span class="metric-value">{{ sampleData[selectedSystem + '-b']?.current }}A</span>
                </div>
                <div class="metric-card">
                  <span class="metric-label">Power Factor</span>
                  <span class="metric-value">{{ (Math.random() * 0.3 + 0.75).toFixed(2) }}</span>
                </div>
                <div class="metric-card">
                  <span class="metric-label">Frequency</span>
                  <span class="metric-value">{{ sampleData[selectedSystem + '-b']?.frequency }}Hz</span>
                </div>
              </div>

              <div v-if="activeTab === 'control'" class="control-buttons">
                <button
                  v-if="isAdmin"
                  class="btn-acb btn-acb-open"
                  @click="controlACB('open', selectedSystem + '-b')"
                  :disabled="acbStatus[selectedSystem + '-b'] === 'OPEN'"
                >
                  Open
                </button>
                <button
                  v-if="isAdmin"
                  class="btn-acb btn-acb-trip"
                  @click="controlACB('trip', selectedSystem + '-b')"
                >
                  Trip
                </button>
                <button
                  v-if="isAdmin"
                  class="btn-acb btn-acb-close"
                  @click="controlACB('close', selectedSystem + '-b')"
                  :disabled="acbStatus[selectedSystem + '-b'] === 'CLOSED'"
                >
                  Close
                </button>
                <div v-if="!isAdmin" class="permission-notice">Only Admin can control ACB</div>
              </div>
              <div v-else class="control-disabled">Monitoring Mode - Control Disabled</div>
            </div>
          </div>

          <!-- Mode Toggle -->
          <div class="mode-toggle">
            <button
              class="mode-btn"
              :class="{ active: activeTab === 'monitoring' }"
              @click="activeTab = 'monitoring'"
            >
              📊 Monitoring Mode
            </button>
            <button
              class="mode-btn"
              :class="{ active: activeTab === 'control' }"
              @click="activeTab = 'control'"
            >
              🎛️ Control Mode
            </button>
          </div>
        </div>

        <!-- Logs View -->
        <div v-if="currentView === 'logs'" class="logs-view">
          <h2 class="page-title">System Logs</h2>
          <p class="page-desc">Centralized log history of all systems and events</p>

          <div class="logs-header">
            <div class="stats-row">
              <div class="stat">
                <div class="stat-label">TOTAL ENTRIES</div>
                <div class="stat-value">{{ logStats.total }}</div>
              </div>
              <div class="stat">
                <div class="stat-label">ALARMS</div>
                <div class="stat-value alarm">{{ logStats.alarms }}</div>
              </div>
              <div class="stat">
                <div class="stat-label">NORMAL</div>
                <div class="stat-value">{{ logStats.normal }}</div>
              </div>
              <div class="stat">
                <div class="stat-label">SYSTEMS AFFECTED</div>
                <div class="stat-value">{{ logStats.rooms }}</div>
              </div>
            </div>

            <div class="logs-actions">
              <button class="btn-export" @click="exportCSV">Export CSV</button>
              <button class="btn-export" @click="exportExcel">Export Excel</button>
              <button class="btn-clear" @click="clearLogs">Clear Logs</button>
            </div>
          </div>

          <div class="filters">
            <button class="filter-btn" :class="{ active: logFilter === 'all' }" @click="logFilter = 'all'">All</button>
            <button class="filter-btn" :class="{ active: logFilter === 'alarms' }" @click="logFilter = 'alarms'">Alarms</button>
            <button class="filter-btn" :class="{ active: logFilter === 'normal' }" @click="logFilter = 'normal'">Normal</button>
          </div>

          <table class="logs-table">
            <thead>
              <tr>
                <th>Time</th>
                <th>System</th>
                <th>Status</th>
                <th>Details</th>
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
            <h2 class="threshold-title">Threshold Configuration</h2>
            <p class="threshold-desc">Edit alarm thresholds for system metrics</p>

            <div class="threshold-actions">
              <button class="btn-reset" @click="resetThresholds">Reset to Defaults</button>
              <button class="btn-save" @click="saveThresholds">Save Changes</button>
            </div>

            <div class="threshold-grid">
              <div v-for="(values, key) in thresholds" :key="key" class="threshold-item">
                <div class="threshold-label">{{ key.toUpperCase() }}</div>
                <div class="threshold-fields">
                  <div class="field">
                    <label>Low</label>
                    <input v-model.number="values.low" type="number" step="0.1">
                  </div>
                  <div class="field">
                    <label>Base</label>
                    <input v-model.number="values.base" type="number" step="0.1">
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

        <!-- Default: Select a System -->
        <div v-if="!selectedSystem && currentView === 'detail'" class="empty-state">
          <div class="empty-icon">🖥️</div>
          <h2>Select a System to Monitor</h2>
          <p>Choose a system from the left sidebar to begin monitoring and control operations</p>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.dashboard-hmi {
  width: 100%;
  min-height: 100vh;
  background: #0f1419;
  display: flex;
  flex-direction: column;
  font-family: 'Inter', 'Roboto', system-ui, sans-serif;
}

/* Header */
.header-top {
  background: linear-gradient(135deg, #0f1419 0%, #1a2332 100%);
  border-bottom: 2px solid rgba(76, 219, 189, 0.2);
  padding: 16px 32px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 32px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
}

.header-left {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.header-brand {
  font-size: 11px;
  letter-spacing: 2.5px;
  color: #4cdbbd;
  text-transform: uppercase;
  font-weight: 700;
}

.header-title {
  font-size: 20px;
  font-weight: 600;
  color: #ffffff;
}

.header-right {
  display: flex;
  align-items: center;
  gap: 20px;
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
  margin-left: 12px;
}

.user-status strong {
  color: #4cdbbd;
}

.user-status-detail {
  font-size: 12px;
  color: #a0aec0;
}

.user-status-detail strong {
  color: #ffffff;
}


/* HMI Container */
.hmi-container {
  display: flex;
  flex: 1;
  overflow: hidden;
  gap: 0;
}

/* Sidebar */
.sidebar {
  width: 200px;
  background: linear-gradient(180deg, #1a2332 0%, #0f1419 100%);
  border-right: 2px solid rgba(76, 219, 189, 0.15);
  padding: 24px 16px;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.sidebar-title {
  font-size: 12px;
  letter-spacing: 1px;
  color: #4cdbbd;
  text-transform: uppercase;
  font-weight: 700;
  padding: 8px 12px;
}

.nav-buttons {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.nav-btn {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 14px 12px;
  background: transparent;
  border: 1.5px solid transparent;
  border-radius: 10px;
  color: #a0aec0;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
}

.nav-btn:hover {
  background: rgba(76, 219, 189, 0.1);
  color: #4cdbbd;
}

.nav-btn.active {
  background: rgba(76, 219, 189, 0.15);
  color: #4cdbbd;
  border-color: #4cdbbd;
  box-shadow: 0 4px 12px rgba(76, 219, 189, 0.2);
}

.nav-icon {
  font-size: 18px;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 24px;
}

.nav-label {
  flex: 1;
  text-align: left;
}

.sidebar-divider {
  height: 1px;
  background: rgba(76, 219, 189, 0.15);
}

.sidebar-section {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.sidebar-label {
  font-size: 11px;
  letter-spacing: 1px;
  color: #a0aec0;
  text-transform: uppercase;
  font-weight: 700;
  padding: 8px 12px;
}

/* Main Content Area */
.main-content {
  flex: 1;
  background: #0f1419;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
}

/* Monitoring View */
.monitoring-view {
  padding: 32px;
  display: flex;
  flex-direction: column;
  gap: 24px;
  overflow-y: auto;
  flex: 1;
}

.view-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 20px;
  margin-bottom: 12px;
}

.view-title {
  font-size: 28px;
  font-weight: 600;
  color: #ffffff;
  margin: 0 0 8px 0;
}

.view-desc {
  font-size: 14px;
  color: #a0aec0;
  margin: 0;
}

.mode-indicator {
  padding: 8px 16px;
  background: rgba(76, 219, 189, 0.15);
  border-radius: 8px;
  font-size: 12px;
  font-weight: 600;
  color: #a0aec0;
  letter-spacing: 0.5px;
}

.mode-indicator.active {
  background: rgba(76, 219, 189, 0.2);
  color: #4cdbbd;
}

/* Power Meter Panel */
.power-meter-panel {
  background: linear-gradient(135deg, rgba(26, 35, 50, 0.8) 0%, rgba(15, 20, 25, 0.8) 100%);
  border: 1px solid rgba(76, 219, 189, 0.2);
  border-radius: 16px;
  padding: 24px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
}

.power-meter-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.power-meter-header h3 {
  font-size: 16px;
  font-weight: 600;
  color: #ffffff;
  margin: 0;
}

.status-badge {
  padding: 6px 14px;
  border-radius: 20px;
  font-size: 12px;
  font-weight: 600;
  letter-spacing: 0.4px;
  text-transform: uppercase;
}

.status-badge.normal {
  background: rgba(16, 185, 129, 0.2);
  color: #22c55e;
}

.status-badge.warning {
  background: rgba(245, 158, 11, 0.2);
  color: #f59e0b;
}

.status-badge.critical {
  background: rgba(239, 68, 68, 0.2);
  color: #ef4444;
}

.meter-display {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
  gap: 16px;
}

.meter-item {
  background: rgba(26, 35, 50, 0.6);
  border: 1px solid rgba(76, 219, 189, 0.15);
  border-radius: 12px;
  padding: 16px;
  text-align: center;
  position: relative;
}

.meter-icon {
  font-size: 28px;
  margin-bottom: 8px;
}

.meter-label {
  font-size: 11px;
  color: #a0aec0;
  text-transform: uppercase;
  letter-spacing: 0.4px;
  margin-bottom: 8px;
  font-weight: 600;
}

.meter-value {
  font-size: 24px;
  font-weight: 700;
  color: #4cdbbd;
}


/* ACB Control Panel */
.acb-control-panel {
  background: linear-gradient(135deg, rgba(26, 35, 50, 0.8) 0%, rgba(15, 20, 25, 0.8) 100%);
  border: 1px solid rgba(76, 219, 189, 0.2);
  border-radius: 16px;
  padding: 24px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
}

.acb-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24px;
  padding-bottom: 16px;
  border-bottom: 1px solid rgba(76, 219, 189, 0.15);
}

.acb-header h3 {
  font-size: 16px;
  font-weight: 600;
  color: #ffffff;
  margin: 0;
}

.btn-check-alert {
  padding: 8px 16px;
  background: rgba(249, 115, 22, 0.15);
  border: 1.5px solid rgba(249, 115, 22, 0.4);
  border-radius: 8px;
  color: #f97316;
  cursor: pointer;
  font-size: 13px;
  font-weight: 600;
  transition: all 0.3s ease;
}

.btn-check-alert:hover {
  background: rgba(249, 115, 22, 0.25);
  border-color: #f97316;
  box-shadow: 0 4px 12px rgba(249, 115, 22, 0.2);
}

/* ACB Channels */
.acb-channel {
  background: rgba(26, 35, 50, 0.6);
  border: 1px solid rgba(76, 219, 189, 0.15);
  border-radius: 12px;
  padding: 20px;
  margin-bottom: 16px;
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
}

.acb-channel:last-child {
  margin-bottom: 0;
}

.channel-info {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.channel-name {
  font-size: 14px;
  font-weight: 600;
  color: #ffffff;
}

.acb-status {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: fit-content;
  padding: 6px 12px;
  border-radius: 8px;
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 0.5px;
  text-transform: uppercase;
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

.channel-metrics {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 12px;
}

.metric-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  padding: 12px;
  background: rgba(26, 35, 50, 0.6);
  border: 1px solid rgba(76, 219, 189, 0.1);
  border-radius: 10px;
}

.metric-label {
  font-size: 10px;
  color: #a0aec0;
  text-transform: uppercase;
  letter-spacing: 0.3px;
  font-weight: 600;
}

.metric-value {
  font-size: 16px;
  font-weight: 700;
  color: #4cdbbd;
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
  align-items: center;
  grid-column: 2;
  flex-direction: column;
  justify-content: center;
}

.btn-acb {
  flex: 1;
  padding: 12px 16px;
  border: none;
  border-radius: 8px;
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  letter-spacing: 0.3px;
  text-transform: uppercase;
  width: 100%;
  box-shadow: 5px 5px 15px rgba(209, 217, 230, 0.3), -3px -3px 10px rgba(255, 255, 255, 0.6);
}

.btn-acb:active {
  transform: scale(0.98);
}

.btn-acb:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.btn-acb-open {
  background: rgba(76, 219, 189, 0.2);
  color: #4cdbbd;
  border: 1px solid #4cdbbd;
}

.btn-acb-open:hover:not(:disabled) {
  background: rgba(76, 219, 189, 0.3);
  box-shadow: 0 4px 12px rgba(76, 219, 189, 0.2);
}

.btn-acb-trip {
  background: rgba(249, 115, 22, 0.2);
  color: #f97316;
  border: 1px solid #f97316;
}

.btn-acb-trip:hover {
  background: rgba(249, 115, 22, 0.3);
  box-shadow: 0 4px 12px rgba(249, 115, 22, 0.2);
}

.btn-acb-close {
  background: rgba(239, 68, 68, 0.2);
  color: #ef4444;
  border: 1px solid #ef4444;
}

.btn-acb-close:hover:not(:disabled) {
  background: rgba(239, 68, 68, 0.3);
  box-shadow: 0 4px 12px rgba(239, 68, 68, 0.2);
}

.control-disabled {
  grid-column: 2;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 16px;
  background: rgba(249, 115, 22, 0.1);
  border: 1px solid rgba(249, 115, 22, 0.3);
  border-radius: 8px;
  color: #f97316;
  font-size: 12px;
  font-weight: 600;
  text-align: center;
}

.permission-notice {
  grid-column: 2;
  padding: 12px;
  background: rgba(245, 158, 11, 0.1);
  border: 1.5px solid #F59E0B;
  border-radius: 8px;
  color: #D97706;
  font-size: 12px;
  font-weight: 600;
  text-align: center;
}

/* Mode Toggle */
.mode-toggle {
  display: flex;
  gap: 12px;
  margin-top: 16px;
}

.mode-btn {
  flex: 1;
  padding: 12px 20px;
  background: rgba(76, 219, 189, 0.1);
  border: 1.5px solid rgba(76, 219, 189, 0.3);
  border-radius: 8px;
  color: #a0aec0;
  cursor: pointer;
  font-size: 13px;
  font-weight: 600;
  transition: all 0.3s ease;
}

.mode-btn:hover {
  background: rgba(76, 219, 189, 0.15);
  border-color: #4cdbbd;
}

.mode-btn.active {
  background: rgba(76, 219, 189, 0.2);
  border-color: #4cdbbd;
  color: #4cdbbd;
  box-shadow: 0 4px 12px rgba(76, 219, 189, 0.2);
}

/* Empty State */
.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 16px;
  flex: 1;
  padding: 40px;
  text-align: center;
}

.empty-icon {
  font-size: 64px;
  opacity: 0.4;
}

.empty-state h2 {
  font-size: 24px;
  color: #ffffff;
  margin: 0;
}

.empty-state p {
  font-size: 14px;
  color: #a0aec0;
  margin: 0;
  max-width: 400px;
}

/* Logs View */
.logs-view {
  padding: 32px;
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.page-title {
  font-size: 28px;
  font-weight: 600;
  color: #ffffff;
  margin-bottom: 8px;
}

.page-desc {
  font-size: 14px;
  color: #a0aec0;
  margin-bottom: 12px;
}

.logs-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 20px;
  margin-bottom: 24px;
  flex-wrap: wrap;
}

.stats-row {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 16px;
  flex: 1;
  min-width: 500px;
}

.stat {
  background: rgba(26, 35, 50, 0.6);
  border: 1px solid rgba(76, 219, 189, 0.2);
  border-radius: 12px;
  padding: 20px;
  text-align: center;
}

.stat-label {
  font-size: 11px;
  color: #a0aec0;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: 12px;
  font-weight: 700;
}

.stat-value {
  font-size: 32px;
  font-weight: 700;
  color: #4cdbbd;
}

.stat-value.alarm {
  color: #ef4444;
}

.logs-actions {
  display: flex;
  gap: 12px;
  flex-direction: column;
  min-width: 140px;
}

.btn-export {
  padding: 10px 20px;
  background: rgba(76, 219, 189, 0.2);
  border: 1px solid #4cdbbd;
  border-radius: 8px;
  color: #4cdbbd;
  cursor: pointer;
  font-size: 13px;
  font-weight: 600;
  transition: all 0.3s ease;
  white-space: nowrap;
}

.btn-export:hover {
  background: rgba(76, 219, 189, 0.3);
  box-shadow: 0 4px 12px rgba(76, 219, 189, 0.2);
}

.btn-clear {
  padding: 10px 20px;
  background: rgba(239, 68, 68, 0.2);
  border: 1px solid #ef4444;
  border-radius: 8px;
  color: #ef4444;
  cursor: pointer;
  font-size: 13px;
  font-weight: 600;
  transition: all 0.3s ease;
  white-space: nowrap;
}

.btn-clear:hover {
  background: rgba(239, 68, 68, 0.3);
  box-shadow: 0 4px 12px rgba(239, 68, 68, 0.2);
}

.filters {
  display: flex;
  gap: 12px;
  margin-bottom: 20px;
  flex-wrap: wrap;
}

.filter-btn {
  padding: 8px 16px;
  background: rgba(76, 219, 189, 0.1);
  border: 1px solid rgba(76, 219, 189, 0.3);
  border-radius: 8px;
  color: #a0aec0;
  cursor: pointer;
  font-size: 13px;
  font-weight: 500;
  transition: all 0.3s ease;
}

.filter-btn:hover {
  border-color: #4cdbbd;
  color: #4cdbbd;
}

.filter-btn.active {
  background: #4cdbbd;
  color: #0f1419;
  border-color: #4cdbbd;
}

.logs-table {
  width: 100%;
  border-collapse: collapse;
  background: linear-gradient(135deg, rgba(26, 35, 50, 0.6) 0%, rgba(15, 20, 25, 0.6) 100%);
  border: 1px solid rgba(76, 219, 189, 0.2);
  border-radius: 12px;
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
  padding: 32px;
  display: flex;
  flex-direction: column;
  gap: 24px;
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

@media (max-width: 1200px) {
  .acb-channel {
    grid-template-columns: 1fr;
  }

  .channel-metrics {
    grid-template-columns: repeat(2, 1fr);
  }

  .control-buttons {
    grid-column: unset;
    flex-direction: row;
  }

  .control-disabled {
    grid-column: unset;
  }
}

@media (max-width: 1024px) {
  .header-top {
    flex-direction: column;
    gap: 16px;
    padding: 12px 20px;
  }

  .sidebar {
    width: 160px;
    padding: 16px 12px;
  }

  .monitoring-view {
    padding: 20px;
  }

  .stats-row {
    grid-template-columns: repeat(2, 1fr);
  }

  .meter-display {
    grid-template-columns: repeat(2, 1fr);
  }

  .threshold-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 768px) {
  .hmi-container {
    flex-direction: column;
  }

  .sidebar {
    width: 100%;
    flex-direction: row;
    overflow-x: auto;
    padding: 12px;
    gap: 8px;
    border-right: none;
    border-bottom: 2px solid rgba(42, 75, 124, 0.1);
  }

  .sidebar-title,
  .sidebar-label {
    display: none;
  }

  .nav-buttons {
    flex-direction: row;
    gap: 6px;
  }

  .sidebar-divider {
    display: none;
  }

  .sidebar-section {
    flex-direction: row;
    gap: 6px;
  }

  .main-content {
    overflow-x: hidden;
  }

  .monitoring-view {
    padding: 16px;
  }

  .power-meter-panel,
  .acb-control-panel {
    padding: 16px;
  }

  .acb-channel {
    grid-template-columns: 1fr;
  }

  .control-buttons {
    flex-direction: row;
  }

  .logs-view,
  .thresholds-view {
    padding: 16px;
  }

  .stats-row {
    grid-template-columns: repeat(2, 1fr);
  }

  .meter-display {
    grid-template-columns: 1fr;
  }

  .threshold-grid {
    grid-template-columns: 1fr;
  }

  .logs-header {
    flex-direction: column;
  }

  .logs-actions {
    flex-direction: row;
  }

  .mode-toggle {
    flex-direction: column;
  }
}
</style>
