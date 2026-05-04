# Elder Care Navigator — Product Requirements Document

**Version:** 3.0  
**Date:** 2026-05-04  
**Author:** Nalini Bateman / Claude  
**Status:** V3 Complete, V4 Planning

---

## 1. Product Overview

### 1.1 Purpose
A single-page HTML tool to help families navigating dementia care options in Toronto. Consolidates facility research, government entitlements, home care options, and family coordination into one shareable resource.

### 1.2 Target Users
- Primary: Family members researching care options for a loved one with dementia
- Secondary: Caregivers coordinating across multiple family members
- Context: Users are typically stressed, time-poor, and new to elder care systems

### 1.3 Design Principles
- **Single file** — No server required, works offline, easy to share
- **Mobile-friendly** — Responsive design for use during facility tours
- **Persistent** — User data saved to browser localStorage
- **Non-judgmental** — Supportive tone throughout

---

## 2. V3 Features (Current)

### 2.1 Navigation (9 Tabs)
1. **Facilities** — Browse 22 Toronto/GTA facilities with ratings, costs, pros/cons
2. **My Tracker** — Track status, notes, and costs for each facility
3. **What Next** — 12-question assessment generating prioritized action list
4. **Stages** — Dementia stage information (dynamic based on assessment)
5. **Home Care** — Agencies, adult day programs, "when home care isn't enough" checklist
6. **Entitlements** — Government programs (Ontario, Federal, City)
7. **Resources** — Equipment suppliers, support organizations
8. **Share** — Tour Mode, checklists, family reference card
9. **Evaluate** — Weighted scoring system for facility comparison

### 2.2 Key Features

#### Facility Cards
- Name, location, phone (click-to-call), website, inspection report links
- Cost with verification status (estimated vs. confirmed)
- Source badges (Friend, Brother, Both, Mine, Exploring)
- Tags, pros, cons, couples policy, waitlist info
- Collapsible sections: Facility Details (from website), Family Feedback & Notes

#### Tour Mode (Share Tab)
- Select facility from dropdown
- Star rating (1-5)
- Quick impressions textarea
- Status selection (Standout, Liked, Questions, Ruled Out)
- Cost quoted field
- Saves directly to Tracker

#### Dynamic Stage Assessment
- Stage selected in "What Next" tab
- Displays on Stages tab with contextual guidance
- Color-coded by severity

#### Priority Generator
- 12 input questions across two categories
- Generates personalized action list
- Considers: stage, behaviours, night care, budget, POA status, etc.

### 2.3 Data Storage (localStorage)
| Key | Contents |
|-----|----------|
| `ecn_facs` | User-added facilities |
| `ecn_costs` | Verified cost updates |
| `ecn_notes` | Facility notes |
| `ecn_family` | Family reference card data |
| `ecn_decision` | What Next assessment answers |
| `ecn_fac_updates` | Status/score updates for built-in facilities |

### 2.4 Facility Data
- 22 facilities across 3 tiers
- Tier 1: Public/Not-for-Profit LTC (~$2,000-$2,900/mo)
- Tier 2: Private Retirement (~$4,500-$10,000/mo)
- Tier 3: Premium Private ($7,500-$20,000+/mo)
- Sources: Friend (ground-level research), Brother (physician curated), Both, Exploring

---

## 3. V4 Requirements (Phase 2)

### 3.1 Tour Date Field
**Priority:** Low  
**Complexity:** Low  
**Description:** Add date picker to Tour Mode so users can record when they visited.

### 3.2 Persist Evaluate Rankings
**Priority:** Medium  
**Complexity:** Low  
**Description:** Save weighted evaluation rankings to localStorage. Currently lost on page refresh.

### 3.3 Multiple Patient Profiles
**Priority:** High  
**Complexity:** Medium  
**Description:** Ability to manage care research for multiple loved ones (e.g., Dad and Mum).

**Requirements:**
- Patient selector dropdown in header
- Separate localStorage namespace per patient
- Switch between profiles without losing data
- Default profile for new users

### 3.4 Shared Family Notes (Backend Required)
**Priority:** High  
**Complexity:** High  
**Description:** Allow family members to see and edit shared notes in real-time.

**Technical Options:**

| Option | Pros | Cons |
|--------|------|------|
| **Firebase (Google)** | Free tier, easy setup, real-time sync | Requires Google accounts |
| **SharePoint List (Microsoft 365)** | Uses existing M365 subscription, enterprise-grade | More complex API (Microsoft Graph) |
| **Supabase** | Open source, PostgreSQL | Requires account setup |

**Recommended:** SharePoint List via Microsoft Graph API
- Nalini already has M365 Business subscription
- Family can be invited as guests
- Keeps data in controlled ecosystem

**Implementation Requirements:**
- Microsoft Entra ID app registration
- OAuth2 authentication flow
- SharePoint List schema for notes
- Read/write via Microsoft Graph API
- Offline queue for sync conflicts

---

## 4. Technical Specifications

### 4.1 Current Architecture
```
Single HTML file (~1600 lines)
├── CSS (embedded, ~300 lines)
│   └── Responsive breakpoints at 700px, 640px
├── HTML structure (9 tab containers)
└── JavaScript (embedded, ~800 lines)
    ├── FACS array (22 facility objects)
    ├── Render functions
    ├── localStorage persistence
    └── Event handlers
```

### 4.2 Facility Object Schema
```javascript
{
  id: 'c1',                    // Unique identifier
  n: 'Facility Name',          // Display name
  loc: 'Address',              // Location
  ph: '416-XXX-XXXX',          // Phone
  web: 'https://...',          // Inspection report URL
  site: 'https://...',         // Facility website
  cost: '$X,XXX-$X,XXX/mo',    // Cost range
  tier: 1,                     // 1=Public, 2=Private, 3=Premium
  src: 'fr',                   // Source: fr/br/bo/me/ex
  status: 'pend',              // star/like/qx/no/pend/booked
  score: 0,                    // Family rating 0-5
  bstar: null,                 // Brother rating (decimal)
  beds: '###',                 // Bed count
  wl: 'Waitlist info',         // Waitlist description
  wc: 'wc-l',                  // Waitlist class: wc-l/wc-m/wc-s/wc-o
  tags: [],                    // Feature tags
  pros: [],                    // Strengths
  cons: [],                    // Concerns
  cpl: [],                     // Couples policy
  notes: '',                   // Default notes
  details: ''                  // HTML from website research
}
```

### 4.3 Hosting
**Recommended:** GitHub Pages
- Free hosting
- Simple deployment (upload index.html)
- Unlisted URL (not indexed by search engines)
- No backend required for V3

---

## 5. User Flows

### 5.1 First-Time User
1. Open link shared by family member
2. Browse Facilities tab to understand options
3. Go to What Next tab, answer assessment questions
4. View generated priority list
5. Review Stages tab for context
6. Check Entitlements for financial programs

### 5.2 During Facility Tour
1. Open Share tab on phone
2. Select facility in Tour Mode
3. Rate with stars, add impressions
4. Set status (Liked/Questions/Ruled Out)
5. Enter quoted cost if provided
6. Save to Tracker

### 5.3 Family Coordination (V4)
1. Sign in with Microsoft account
2. View shared notes from other family members
3. Add own observations
4. See real-time updates

---

## 6. Success Metrics

- Families can compare facilities without spreadsheets
- Tour impressions captured on mobile during visits
- Priority actions clear after assessment
- Entitlements discovered that would otherwise be missed
- Family members aligned on information (V4)

---

## 7. Out of Scope (Future Phases)

- Automated waitlist tracking
- Direct facility booking
- Integration with Ontario Health atHome systems
- Caregiver scheduling
- Medical record storage
- Payment processing

---

## 8. File Locations

| File | Location |
|------|----------|
| V3 HTML | `/Users/nalinibateman/Downloads/toronto_elder_care_navigator_v3.html` |
| PRD | `/Users/nalinibateman/Library/CloudStorage/OneDrive-AmanziConsultingInc/AmanziFlow/Claude-Workspace/elder-care-navigator-prd.md` |
| Session Summary | `/Users/nalinibateman/Library/CloudStorage/OneDrive-AmanziConsultingInc/AmanziFlow/CLAUDE/Session Summaries/2026-05-04_elder-care-navigator-v3.md` |

---

## 9. Revision History

| Version | Date | Changes |
|---------|------|---------|
| 3.0 | 2026-05-04 | Initial PRD created after V3 completion |
| 3.0.1 | 2026-05-04 | Alphabetized all facility dropdowns (Evaluate tab, Tour Mode) |
