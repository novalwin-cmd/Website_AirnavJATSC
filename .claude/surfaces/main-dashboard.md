# Surface Brief: Main Dashboard

## Job and Audience

**Operator**: Admin user in control room or remote location, opening the dashboard to monitor real-time power system state and execute breaker control actions. Needs immediate visibility of all generator and panel metrics, with confidence in every action taken. 

**Supervisor/Monitor**: Engineering user (read-only) viewing the same dashboard to oversee system status without control capability. Same data, but control buttons are visually disabled.

**Visitor mode**: Operate — task completion under time pressure; every element must reduce cognitive load and prevent errors.

## Outcome and Proof

**Primary outcome**: Operator sees current state of both generator sets and outgoing power panels in a single unified interface, can switch between focused views (generators vs. panels), detects alerts immediately via visual + audio cues, acknowledges anomalies, and executes breaker control actions with secondary confirmation.

**Success proof**:
- Real-time metrics visible within sub-second latency (voltage, current, frequency, power factor, power consumption)
- Threshold breaches trigger audio/visual interrupt and visible alert badge on the affected metric
- Admin can switch breaker states (Open/Closed/Reclose) with confirmation dialog; engineering users see buttons disabled but not hidden
- Operator navigates to Logs/Thresholds/Export without leaving the main dashboard view
- All actions (breaker switches, threshold changes, exports) logged with timestamp and user attribution

## Selected Direction

**Visual authority**: Industrial/technical aesthetic from PRODUCT.md. Data clarity over decoration. Color used strategically for status (breaker state badges, alert indicators). Recall reference screenshots provided by stakeholder for control room conventions.

**Structural thesis**: Single unified dashboard with tabbed/toggled view. Users switch between "Generator Sets" and "Power Panels" tabs to focus on specific subsystems. Header contains user info, mode toggle (Monitoring/Control), action buttons (Logs, Thresholds, Export), and logout. Body displays the selected tab's content. Secondary surfaces (Logs, Thresholds, Export) slide in as side panels or modals without full navigation away.

**Sequence**: 
1. Login → Main dashboard showing all subsystems (generators + panels unified)
2. Real-time metrics stream and update sub-second for all subsystems
3. Alert triggers → visual badge on affected metric + audio alarm + user acknowledges
4. User toggles Monitoring/Control Mode (visual feedback: control buttons enabled/disabled)
5. User clicks breaker control button (Open/Closed/Reclose) → confirmation dialog → action logged
6. User clicks Logs/Thresholds/Export → side panel/modal slides in with content
7. User scrolls through all subsystems or dashboard remains visible for continuous monitoring

**Focal moment**: Breaker control action. Admin sees button, clicks, sees confirmation dialog with current state + target state clearly labeled, confirms, sees immediate feedback (button state updates, action appears in log). Engineering user sees the same button but it's disabled/grayed.

## Scope and Boundaries

**Fidelity**: Production-ready interactive mockup. Not a prototype; must show real data, real states, real confirmations.

**Breadth**: 
- Main dashboard (login → authenticated view)
- Unified dashboard view: all four subsystems (DSE Technical, Priority, A7, T7) displayed with identical structure
- Each subsystem shows: 5 metric cards (V/A/Hz/Power Factor/kW) + control buttons (Open/Closed/Reclose)
- Secondary surfaces: Logs viewer, Thresholds configurator, Export dialog
- Alert system (visual + audio feedback, acknowledgment UI)

**Interactivity**: Full workflow interactions required. All buttons, toggles, and forms must show expected feedback and state changes. Confirmation dialogs for destructive actions (breaker switching). Real-time metric updates shown with animation or visual refresh.

**Named target**: "Main Dashboard" surface only. Does NOT include:
- Tab navigation (single unified view of all subsystems; no switching between sections)
- Backend data integration (assume data flows correctly from power meters)
- Authentication server implementation (login form only, placeholder auth)
- Actual file export (export dialog shows format options; actual download is backend responsibility)
- Mobile-first design (desktop-first; responsive layout secondary)

**Untouched**:
- Exact threshold values (UI allows setting; defaults are backend configuration)
- Alert escalation rules (visual + audio is minimum; escalation logic undefined)
- Real-time data source (assume WebSocket or polling works; UI agnostic to mechanism)

**Anti-goals**: 
- Do NOT add decorative animations; only functional feedback (button state, metric updates, alerts)
- Do NOT design separate "engineer" dashboard; same UI with role-based button state
- Do NOT hide control buttons from engineers; disable them visually
- Do NOT make the UI too complex; industrial aesthetic = clean, scannable, high contrast

## States and Ranges

**Real-time metrics (realistic ranges)**:
- Voltage: 0–500V (ranges vary per panel; A7 vs T7 may differ)
- Current: 0–1000A
- Frequency: 45–65 Hz (standard power grid)
- Power Factor: 0.8–1.0 (leading/lagging)
- Power: 0–500+ kW (depends on load)
- Sparklines: 30–60 seconds of history, updating sub-second

**Breaker states**: Open, Closed, Reclose (three distinct states shown as badges/buttons)

**Alert states**:
- No alert: metric within threshold, normal color
- Warning: metric approaching threshold (yellow/amber badge)
- Critical: metric exceeds threshold (red badge + audio alarm)
- Acknowledged: operator clicked acknowledge; badge persists but alarm silenced

**User states**:
- Logged out (login screen)
- Authenticated as Admin (full control + monitoring)
- Authenticated as Engineering (monitoring only; buttons disabled)
- Monitoring Mode (primary view; control buttons visually disabled but present)
- Control Mode (control buttons enabled; visual indicator shows mode active)

**Modal/panel states**:
- Logs: list of all user actions, logins, alerts, with timestamps and user attribution
- Thresholds: form to set upper/lower limits per metric; shows current values
- Export: dialog showing format options (CSV, PDF), date range selector, export button

## Interaction and Layout

**Header** (always visible):
- Left: Application title / branding
- Center-left: "Signed in as [username]"
- Center-right: Monitoring/Control Mode toggle (radio or switch; visual indication of active mode)
- Right: Buttons for Logs, Thresholds, Export; Logout button

**Main content area** (single unified view, no tabs):
- All four subsystems displayed together, stacked vertically or in a grid
- Subsystems: DSE Technical, Priority (generators) + A7, T7 (power panels)

**Unified subsystem card** (identical structure for all):
- Subsystem name and identifier (e.g., "DSE Technical Generator", "A7 Panel")
- Real-time metrics displayed as individual cards in a row/grid:
  - **Voltage (V)**: current value + unit + sparkline chart (30-60 sec history)
  - **Current (A)**: current value + unit + sparkline chart
  - **Frequency (Hz)**: current value + unit + sparkline chart
  - **Power Factor**: current value (0.0–1.0) + sparkline chart
  - **Power (kW)**: current value + unit + sparkline chart
- Each metric card shows: alert badge if threshold breached (red for critical, yellow for warning)
- Control buttons below metrics: **Open** | **Closed** | **Reclose** (enabled for admin/control mode; disabled for engineering or monitoring mode)
- Status indicator: shows current state (icon/badge showing which button state is active)

**Alert system**:
- Visual: red/amber badge on affected metric or breaker card
- Audio: system beep or alert sound (configurable mute)
- Dismissible: "Acknowledge" button on alert badge or modal that silences alarm but persists alert display

**Confirmations**:
- Breaker control action: modal dialog showing "Confirm: Switch [Breaker] from [Current State] to [Target State]?" with Cancel/Confirm buttons
- After confirmation: immediate visual feedback (button state updates, action logged, brief success message)

**Secondary panels** (Logs, Thresholds, Export):
- Slide in from right edge or appear as modal overlay
- Include close button (X) to return to main dashboard
- Do NOT navigate away; main dashboard remains visible/accessible in background (or lightly dimmed if modal)

**Hierarchy and emphasis**:
- Real-time data (metrics, breaker states) largest and most prominent
- Control buttons clearly visible but not aggressive (secondary emphasis; primary is monitoring)
- Alerts interrupt visually and aurally
- Secondary info (history, configuration) accessed via panel/modal, not always visible

**Responsiveness**:
- Desktop-first (control room workstations with large screens)
- Tablets: layout adapts; tabs may stack or side-by-side adjust
- Mobile: not a priority; focus on desktop and tablet (operators in control room)

**Affordances and feedback**:
- Buttons show hover state (highlight), active/pressed state (darker), disabled state (grayed)
- Real-time metrics show subtle animation on update (color flash, value highlight, or smooth transition)
- Modal/panel transitions: slide-in animation from right edge (smooth, not jarring)
- Confirmation dialogs: modal overlay with clear action buttons

## Constraints and Open Decisions

**Binding constraints**:
- React + TypeScript stack (PRODUCT.md)
- Localhost deployment only (no external internet; assume same network)
- Two user roles: Admin (full access), Engineering (read-only)
- Near real-time updates (sub-second latency required; implies WebSocket or very frequent polling)
- Industrial/technical aesthetic (PRODUCT.md brand commitment)
- WCAG 2.1 AA accessibility (standard web best practices)
- Secondary confirmation required for all breaker control actions
- All actions logged with timestamp and user attribution

**Open decisions** (builder will decide):
- Exact data source/backend API for real-time metrics (UI agnostic)
- Exact audio alert sound (system default, configurable, or custom audio)
- Threshold default values (backend configuration; UI allows override)
- Exact tab toggle implementation (radio buttons, pill tabs, or traditional tabs)
- Sparkline chart library (Recharts, Chart.js, custom SVG, etc.)
- Export formats and column mapping (standard industrial reporting conventions)
- Color palette for status badges (engineer will define tokens; industrial convention suggests green/yellow/red)

**Reusable components**:
- Metric card (value + unit + sparkline + alert badge)
- Breaker state badge (Open/Closed/Reclose indicator)
- Control button (with disabled state for read-only users)
- Alert badge (color-coded warning/critical)
- Confirmation dialog modal
- Secondary panel/modal container
- Real-time data polling hook (React custom hook for WebSocket or polling logic)

**No integration with**:
- Backend database (UI assumes data arrives via API/WebSocket)
- Authentication server (login form only; assume token-based session)
- Email/SMS alerts (audio/visual only)
- External monitoring services

## Next Steps

1. Confirm this brief with stakeholder
2. Designer creates wireframes and visual mockup based on this structure
3. Developer implements React components following this interaction model and state diagram
4. QA verifies all workflows (login, monitoring, alerts, control actions, secondary panels)
