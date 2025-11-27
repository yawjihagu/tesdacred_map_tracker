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
- **Color-coded by type**: TVET (blue #2196F3), HEI (green #4CAF50), College (orange #FF9800), Enterprise (purple #9C27B0)
- **Pulsing animations**: CSS keyframe animations (`@keyframes pulse-blue`, etc.)
- **Interactive popups**: Display institution details with embedded HTML tables for courses

### Filtering System
Four simultaneous filters implemented in `filterInstitutions()` function:
1. **Institution Type** - TVET/HEI/College/Enterprise
2. **Accreditation Status** - Accredited/Pending/Expired
3. **Province** - 7 provinces in BARMM region
4. **Expiring Soon** - Checkbox for 90-day warning (uses `Date` comparison)

### Authentication (`login.html`)
- **Demo credentials** hardcoded in `validUsers` array
- Session storage: `localStorage` (remember me) or `sessionStorage`
- Redirects to `ccdomap.html` (should reference `index.html`)

## Common Development Tasks

### Adding New Institutions
1. Add object to `institutions` array in `index.html` (around line 900)
2. Use existing institution as template
3. Required fields: `name`, `type`, `lat`, `lng`, `province`, `status`, dates
4. Format `coursesImage` as HTML `<table class='courses-table'>` with proper structure

### Updating Provinces
- Modify `provinceCenters` object (line ~895) when adding new provinces
- Add corresponding `<option>` to `#provinceFilter` dropdown (line ~130)
- Update legend colors if adding institution types

### Debugging Map Issues
- Check browser console for Leaflet errors
- Verify coordinates are valid (lat/lng within Philippines bounds)
- Ensure `status` values match exactly: "accredited", "pending", "expired"
- Test with collapsed control panel (`control-panel.collapsed` class)

## Styling Conventions

### CSS Structure (Embedded in `<style>`)
- **Utility-first approach** - All styles in single `<style>` block per file
- **Gradient theme**: Primary color `#1e3c72`, secondary `#2a5298`
- **Responsive breakpoints**: `@media (max-width: 768px)` for mobile
- **Print styles**: Hide controls with `@media print`

### Component Patterns
```css
/* Header with logo + title */
.header { display: flex; gap: 12px; background: linear-gradient(...); }

/* Collapsible panel (control-panel) */
.control-panel.collapsed > *:not(.toggle-panel) { display: none; }

/* Map markers with pulse */
.pulse-marker.tvet { animation: pulse-blue 2s infinite; }
```

## Deployment

### GitHub Pages Configuration
- Deploys from `main` branch via `.github/workflows/static.yml`
- Entire repository published as-is (no build step)
- Image asset: `mbhte logo.png` must be in root directory
- Access at: `https://yawjihagu.github.io/tesdacred_map_tracker/`

### File References
- **Fix login redirect**: Change `'ccdomap.html'` to `'index.html'` in `login.html` (line ~265)
- All asset paths are relative (no `/` prefix needed for same-directory files)

## Data Maintenance

### Accreditation Expiry Logic
Calculated in `filterInstitutions()`:
```javascript
const expiryDate = new Date(institution.expiryDate);
const daysUntilExpiry = Math.ceil((expiryDate - today) / (1000 * 60 * 60 * 24));
```
Institutions expiring in <90 days highlighted when "expiring soon" checked.

### Statistics Calculation
Real-time counts updated on filter changes:
- `#totalCount` - Visible institutions
- `#accreditedCount`, `#pendingCount`, `#expiredCount` - By status
- Located in control panel stats grid

## Common Pitfalls

1. **Hardcoded data only** - No database or API; all changes require code edits
2. **Date format**: Must be parseable by JavaScript `Date()` constructor (e.g., "July 4, 2022")
3. **Province filter keys**: Must match institution `province` field exactly (kebab-case)
4. **Table HTML in strings**: Must escape quotes properly in `coursesImage` field
5. **Login page redirect**: Currently points to non-existent `ccdomap.html`

## Testing Checklist

When making changes:
- [ ] Test all 4 filter combinations
- [ ] Verify map markers render with correct colors
- [ ] Check popup content displays properly
- [ ] Test "expiring soon" filter accuracy
- [ ] Validate mobile responsiveness (768px breakpoint)
- [ ] Confirm search functionality works
- [ ] Test print layout (hides controls)

## Contact & Context

- **Organization**: BARMM TESD (Technical Education & Skills Development)
- **Ministry**: MBHTE (Ministry of Basic, Higher and Technical Education)
- **Geographic Coverage**: 7 provinces - Cotabato City, Maguindanao del Norte/Sur, Lanao del Sur, Basilan, Sulu, Tawi-Tawi
- **Primary Users**: TESD staff tracking institutional accreditation compliance
