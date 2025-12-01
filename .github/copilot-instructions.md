# BARMM TESD Accreditation Map System - AI Coding Agent Instructions

## Project Overview
This is a web-based Geographic Information System (GIS) for tracking and visualizing Technical Vocational Education and Training (TVET) institutions, Higher Education Institutions (HEIs), colleges, and enterprises across the Bangsamoro Autonomous Region in Muslim Mindanao (BARMM). The system maps accreditation status, program offerings, and expiration dates.

## Architecture & Technology Stack

### Frontend-Only Application
- **Pure HTML/CSS/JavaScript** - No build process, frameworks, or dependencies
- **Leaflet.js** (CDN) - Interactive mapping via OpenStreetMap
- **Single-page applications** - `index.html` (main map), `login.html` (authentication)
- **Client-side only** - No backend server; all data hardcoded in JavaScript

### Key Technical Patterns

#### Data Structure (in `index.html`)
```javascript
const institutions = [
  {
    name: "Institution Name",
    type: "tvet|hei|college|enterprise",  // Color-coded on map
    lat: 7.191994504,
    lng: 124.2348899,
    address: "Full address",
    province: "cotabato-city|maguindanao-del-norte|...",  // Filter by province
    status: "accredited|pending|expired",
    accreditationDate: "Month Day, Year",
    expiryDate: "Month Day, Year",
    contactPerson: "Name (Role)",
    contactNumber: "Phone",
    email: "email@domain.com",
    coursesImage: "<table class='courses-table'>..."  // HTML table string
  }
]
```

#### Province Centers for Map Navigation
Located after map initialization in `index.html`:
```javascript
const provinceCenters = {
  "cotabato-city": { lat: 7.2047, lng: 124.2383, zoom: 12 },
  "maguindanao-del-norte": { lat: 7.3000, lng: 124.2500, zoom: 10 },
  // ... other provinces
};
```

## Core Features & Implementation

### Map Markers & Styling
**Color scheme** (defined in CSS `@keyframes` and `.pulse-marker` classes):
- TVET: Blue #2196F3 (`pulse-blue` animation)
- HEI: Green #4CAF50 (`pulse-green`)
- College: Orange #FF9800 (`pulse-orange`)
- Enterprise: Purple #9C27B0 (`pulse-purple`)

**Marker creation** - `createPulsingIcon(type)` function uses Leaflet divIcons:
```javascript
L.divIcon({
  className: `pulse-marker ${type}`,  // Type drives CSS animation
  iconSize: [30, 30],
  iconAnchor: [15, 15]
})
```

**Popup content** - `createPopupContent(inst)` builds HTML with:
- Status badge (colored pill)
- Contact info
- Embedded `coursesImage` table (HTML string rendered directly)
- Action buttons (Report/Certificate - currently alert placeholders)

### Filtering System
`filterInstitutions()` applies 4 simultaneous filters:
1. **Type** (`#typeFilter`) - all/tvet/hei/college/enterprise
2. **Status** (`#statusFilter`) - all/accredited/pending/expired
3. **Province** (`#provinceFilter`) - all + 7 provinces (kebab-case)
4. **Expiring Soon** (`#expiringFilter`) - `isExpiringSoon()` checks `<= 90 days`

**Critical**: All filters use exact string matching (case-sensitive for status/type)

### Authentication (`login.html`)
**Demo credentials** (`validUsers` array ~line 230):
```javascript
{ username: 'admin', password: 'admin123', role: 'Administrator' }
{ username: 'user', password: 'user123', role: 'User' }
{ username: 'tesd', password: 'tesd2024', role: 'TESD Staff' }
```

**Session storage**: 
- `localStorage` if "remember me" checked (30 days)
- `sessionStorage` otherwise (session only)
- Both store JSON: `{username, fullName, role, loginTime, expiresIn}`

**Login flow**: Successfully redirects to `index.html` after authentication

## Common Development Tasks

### Adding New Institutions
1. **Locate data array**: `index.html` ~line 900-2500 (`const institutions = [...]`)
2. **Copy existing entry** as template (ensure all fields present)
3. **Required fields**:
   - `name`, `type`, `lat`, `lng`, `address`, `province`, `status`
   - `accreditationDate`, `expiryDate` (must be parseable by `new Date()`)
   - `contactPerson`, `contactNumber`, `email`
   - `coursesImage` - HTML string with `<table class='courses-table'>` structure
4. **Critical validations**:
   - `type` must be: tvet, hei, college, or enterprise (lowercase)
   - `status` must be: accredited, pending, or expired (lowercase)
   - `province` must match key in `provinceCenters` (kebab-case)
   - Coordinates must be within BARMM bounds (lat: 4.8-8.1, lng: 119.5-125.0)

### Updating Provinces
**Three files to modify**:
1. `provinceCenters` object (~line 895) - add `{lat, lng, zoom}`
2. `#provinceFilter` dropdown (~line 130) - add `<option value="key">Display Name</option>`
3. Test navigation: Selecting province should pan/zoom to center

**Note**: Province keys use kebab-case (e.g., `"maguindanao-del-norte"`)

### Updating Login Credentials
**Location**: `login.html` ~line 230 (`validUsers` array)
**Format**: `{ username: 'string', password: 'string', role: 'string' }`
**Warning**: Plaintext passwords - this is demo auth only, not production-secure

### Debugging Map Issues
**Common problems**:
1. **Markers not showing**: Check console for Leaflet errors, verify `lat`/`lng` are numbers
2. **Filters not working**: Ensure exact string match (lowercase, no typos)
3. **Popup HTML broken**: Escape quotes in `coursesImage` table HTML
4. **Province navigation fails**: Verify key in `provinceCenters` matches `institution.province`
5. **Expiry filter wrong**: Check date format is `"Month Day, Year"` (e.g., `"July 4, 2022"`)

## Styling Conventions

### CSS Architecture (Embedded `<style>` blocks)
**No external CSS files** - all styles inline in each HTML file

**Color palette**:
- Primary: `#1e3c72` (dark blue)
- Secondary: `#2a5298` (lighter blue)
- Gradients: `linear-gradient(135deg, #1e3c72 0%, #2a5298 100%)`

**Key CSS patterns**:
```css
/* Collapsible control panel */
.control-panel.collapsed > *:not(.toggle-panel) { display: none; }

/* Pulsing map markers */
.pulse-marker.tvet { animation: pulse-blue 2s infinite; }

/* Status badges */
.badge-accredited { background: #4CAF50; }  /* Green */
.badge-pending { background: #FF9800; }      /* Orange */
.badge-expired { background: #f44336; }      /* Red */
```

**Responsive breakpoint**: `@media (max-width: 768px)` for mobile layout

**Print styles**: `@media print` hides `.control-panel` and `.header-actions`

## Deployment & Workflows

### GitHub Pages (Static Hosting)
**Workflow file**: `.github/workflows/static.yml` (+ duplicate variants static1/2/3.yml)
- Deploys from `main` branch on every push
- No build step - publishes raw HTML/CSS/JS
- Live URL: `https://yawjihagu.github.io/tesdacred_map_tracker/`

**Critical asset**: `mbhte logo.png` must be in repository root (referenced by both HTML files)

**Known issues**:
1. Multiple workflow files (static.yml, static1-3.yml) - likely only static.yml is active

### Local Development
**No build process** - just open files in browser:
1. Open `login.html` in browser
2. Use credentials: `admin`/`admin123` or `tesd`/`tesd2024`
3. Redirects to index.html (if running via file:// protocol, may need local server for session storage)

**For local testing**: `python3 -m http.server 8000` or `npx serve`

## Data Flows & Key Functions

### Filtering Logic (`filterInstitutions()`)
**Called by**: All filter dropdown `onchange` events + checkbox
**Process**:
1. Reads 4 filter values (type, status, province, expiring)
2. Filters `institutions` array with `Array.filter()` - all conditions must pass
3. Calls `addMarkers(filtered)` to update map
4. Updates stats counters via `updateStats(filtered)`

**Expiry calculation** (`isExpiringSoon()`):
```javascript
const diffDays = Math.ceil((new Date(expiryDate) - new Date()) / (1000 * 60 * 60 * 24));
return diffDays <= 90 && diffDays >= 0;
```

### Marker Management
**Rendering cycle**:
1. `addMarkers(institutions)` - clears old markers, creates new ones
2. `createPulsingIcon(type)` - generates Leaflet divIcon with CSS class
3. `createPopupContent(inst)` - builds HTML popup string
4. Binds popup + tooltip to each marker

**Current markers tracking**: `currentMarkers` array (for cleanup on re-filter)

### Statistics (`updateStats()`)
Real-time counts in control panel:
- `#totalCount` - `institutionsToShow.length`
- `#accreditedCount` - `filter(i => i.status === 'accredited').length`
- `#pendingCount`, `#expiredCount` - same pattern

### Search (`#searchInput` listener)
**Triggers**: On `input` event (live search)
**Behavior**:
- Minimum 2 characters
- Searches `name` and `address` fields (case-insensitive)
- Displays results in `#searchResults` dropdown
- Clicking result calls `zoomToInstitution(name)` and opens popup

## Common Pitfalls & Gotchas

1. **No database** - All data in `institutions` array. Adding/editing requires code commit + deploy.
2. **Date parsing sensitivity** - Format must be `"Month Day, Year"` (e.g., `"July 4, 2022"`). Invalid dates break expiry filter.
3. **Case-sensitive matching** - `type` and `status` must be lowercase (tvet, not TVET)
4. **Province key mismatch** - `institution.province` must exactly match key in `provinceCenters` (kebab-case)
5. **HTML escaping in coursesImage** - Use backticks for template literals or escape quotes in table HTML
6. **Logo fallback** - If `mbhte logo.png` missing, login page shows SVG fallback (data URI)
7. **Marker clustering** - Not implemented. Too many markers in one area will overlap.
8. **No form validation** - Adding institutions via code means no runtime checks for invalid data
9. **Session persistence** - Refreshing page maintains login (localStorage/sessionStorage), but logout clears all sessions

## Testing Checklist

**After any data/code changes**:
- [ ] All 4 filters (type, status, province, expiring) work independently and combined
- [ ] Markers render with correct colors (TVET blue, HEI green, College orange, Enterprise purple)
- [ ] Popups display institution details + courses table correctly
- [ ] "Expiring soon" checkbox shows only institutions ≤90 days from expiry
- [ ] Province dropdown navigates map to correct location
- [ ] Search bar filters and zooms to institutions
- [ ] Statistics counters update on filter changes
- [ ] Login/logout flow works (test both localStorage and sessionStorage)
- [ ] Mobile layout works at 768px breakpoint (control panel resizes)
- [ ] Print layout hides control panel and header actions
- [ ] Export CSV downloads with current filter applied
- [ ] Map/List view toggle switches between modes

**Visual regression**:
- [ ] Pulsing marker animations working
- [ ] Header logo displays (or fallback SVG if missing)
- [ ] Status badges colored correctly (green/orange/red)
- [ ] Collapsible panel toggle button works (◀/▶ icon switches)

## Contact & Context

**Organization**: BARMM TESD (Technical Education & Skills Development)  
**Ministry**: MBHTE (Ministry of Basic, Higher and Technical Education)

**Geographic Coverage** (7 provinces):
1. Cotabato City (region center)
2. Maguindanao del Norte
3. Maguindanao del Sur
4. Lanao del Sur (includes Marawi City)
5. Basilan (includes Lamitan City, Isabela City)
6. Sulu (includes Jolo)
7. Tawi-Tawi (includes Bongao)

**Institution Types**:
- **TVET** - Technical Vocational Education Training centers (most common)
- **HEI** - Higher Education Institutions (colleges/universities)
- **College** - Standalone colleges
- **Enterprise** - Private training providers

**Primary Users**: TESD staff tracking accreditation compliance and expiry dates

## Quick Reference

**File structure**:
```
/
├── index.html          # Main app (3054 lines, ~900-2500 is data)
├── login.html          # Auth gate (783 lines)
├── mbhte logo.png      # Logo asset (must be present)
├── README.md           # Minimal project description
└── .github/
    ├── copilot-instructions.md  # This file
    └── workflows/
        ├── static.yml   # GitHub Pages deploy (active)
        └── static[1-3].yml  # Duplicate workflows (unused?)
```

**Key functions** (all in `index.html`):
- `filterInstitutions()` - Apply all filters and update map
- `addMarkers(institutions)` - Render/update map markers
- `createPopupContent(inst)` - Build popup HTML
- `isExpiringSoon(date)` - Check if date within 90 days
- `zoomToInstitution(name)` - Pan map to specific institution
- `exportToCSV()` - Download filtered data as CSV
- `switchView(view)` - Toggle between map/list views
- `togglePanel()` - Collapse/expand control panel
- `logout()` - Clear session and redirect to login
