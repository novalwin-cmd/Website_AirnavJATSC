# JATSC Tower - Monitoring dan Kontrol Panel

A modern monitoring and control dashboard for JATSC Tower electrical systems, built with Vue 3 + Vite.

## Features

### 🎯 4 Main Systems
1. **DSE 1** - Distribution System Equivalent 1 (Monitoring)
2. **DSE 2** - Distribution System Equivalent 2 (Monitoring)
3. **Panel Powermeter 1** - Power Meter with ACB Control
4. **Panel Powermeter 2** - Power Meter with ACB Control

### 📊 Real-time Monitoring
- **Voltage** (V) - Current voltage reading
- **Current** (A) - Current draw
- **Frequency** (Hz) - System frequency
- **Power Factor** / **Power** (kW) - Power metrics

### 🎛️ ACB Controls (Admin Only)
- **Open** - Open the Air Circuit Breaker
- **Trip** - Emergency trip the ACB
- **Close** - Close the ACB

### 📋 Centralized Logging
- Real-time log entries from all systems
- Status indicators (Normal/Alarm)
- Filter by type and room
- Export to CSV/Excel
- Clear logs functionality

### ⚙️ Threshold Management
- Edit alarm thresholds for all metrics
- Save settings locally
- Reset to defaults
- Easy-to-use interface

### 🔐 User Roles
- **Admin** (admin / airnav2026)
  - Full monitoring access
  - Control ACB operations
  - Can modify thresholds
  
- **Engineer** (engineer / monitoring)
  - Monitoring only
  - No control permissions
  - View-only access

## Getting Started

### Prerequisites
- Node.js 16+
- npm or yarn

### Installation

```bash
cd jatsc-monitoring
npm install
```

### Development

```bash
npm run dev
```

Open [http://localhost:5173/](http://localhost:5173/) in your browser.

### Build for Production

```bash
npm run build
```

The built files will be in the `dist/` directory.

### Preview Build

```bash
npm run preview
```

## Project Structure

```
src/
├── components/
│   ├── LoginPage.vue          # Authentication page
│   ├── Dashboard.vue          # Main dashboard container
│   └── dashboard/
│       ├── SystemMenu.vue     # System selection grid
│       ├── MonitoringDetail.vue # Detailed monitoring view
│       ├── LogsPanel.vue      # Centralized logs
│       └── ThresholdsPanel.vue # Threshold settings
├── App.vue                    # Root component
├── main.js                    # Entry point
└── style.css                  # Global styles
```

## Test Credentials

### Admin Account
- **Username**: `admin`
- **Password**: `airnav2026`
- Permissions: Monitoring + Control

### Engineer Account
- **Username**: `engineer`
- **Password**: `monitoring`
- Permissions: Monitoring only

## Key Features

### Monitoring Mode
- View real-time metrics for each system
- Interactive metric cards with visual indicators
- Historical data trends

### ACB Control
- Open/Trip/Close operations
- Real-time status updates
- Action logging

### Logs
- Centralized log management
- Filter by status (Normal/Alarm)
- Export capabilities
- Statistics dashboard

### Thresholds
- Customizable alarm thresholds
- Local storage persistence
- Reset to defaults option

## Customization

### Colors
Edit the theme colors in component `<style>` sections:
- Primary: `#4cdbbd` (Teal)
- Alarm: `#ef4444` (Red)
- Warning: `#f97316` (Orange)
- Info: `#60a5fa` (Blue)

### Data
Modify the `systemsData` object in `Dashboard.vue` to add/remove systems or change metrics.

### Metrics
Update metric configurations in each system object:
```javascript
metrics: [
  { icon: '⚡', label: 'Voltage', value: '230V', color: '#4cdbbd', line: 30 }
]
```

## Browser Support
- Chrome/Edge 90+
- Firefox 88+
- Safari 14+

## License
© 2026 AIRNAV JATSC Tower

## Support
For issues or feature requests, please refer to the project documentation.
