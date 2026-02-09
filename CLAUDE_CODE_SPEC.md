# Franciscan Woods HOA Project Tracker - Claude Code Build Spec

## What This Is

A simple, card-based web app that reads project data from a Google Sheet and displays it as a clean, mobile-friendly dashboard for up to 5 HOA board members. Hosted free on Netlify with password protection.

## Architecture

```
Google Sheet (data) --> Google Sheets API (free) --> Static HTML/JS site --> Netlify (free hosting + password)
```

- **No backend server.** The site is pure HTML/CSS/JS.
- **No database.** Google Sheet IS the database.
- **No user accounts.** Netlify's built-in site password handles access.
- **No build step.** Just static files you drag-drop to Netlify.

## Google Sheet Structure

The Google Sheet has one sheet called "Projects" with these columns (A through R):

| Column | Field | Required | Notes |
|--------|-------|----------|-------|
| A | Project Name | Yes | Unique identifier |
| B | Status | Yes | One of: Planning, Collecting Bids, Scheduled, In Progress, Complete |
| C | Urgency | Yes | One of: Emergency, High, Medium, Low |
| D | Lead | Yes | Board member first name |
| E | Units | Yes | Which units/buildings affected (e.g., "All", "Bldg 1 & 2", "Unit 14") |
| F | Category | Yes | e.g., Roofing, Plumbing, Mechanical, Grounds, Structural |
| G | Description | No | Brief description of the project |
| H | Next Action | Yes | The single next thing that needs to happen |
| I | Next Action Owner | Yes | Who is responsible for the next action |
| J | Next Action Due | Yes | When the next action is due |
| K | Target Date | No | Overall project target completion |
| L | Vendors & Bids | No | Vendor names with bid amounts in parentheses |
| M | Vendor Selected | No | Which vendor was chosen |
| N | Budgeted | No | Dollar amount budgeted |
| O | Approved | No | Dollar amount approved by board |
| P | Actual | No | Actual cost (for completed projects) |
| Q | Fund Source | No | "Reserve" or "Operating" |
| R | Notes | No | Any additional context |

## Google Sheets API Setup

1. Create a Google Cloud project (free)
2. Enable the Google Sheets API
3. Create an API key (restrict it to Sheets API only)
4. The Google Sheet should be set to "Anyone with the link can view" -- this is ONLY so the API key can read it. The link itself is never shared publicly, and the website is password-protected on Netlify.

**Important:** The API key will be visible in the frontend JavaScript. This is acceptable because:
- The key is restricted to only read the Sheets API
- The sheet is read-only via this key
- The Netlify password protects the site itself
- This is a common pattern for free-tier static sites

If the board wants more security later, we can move to a Netlify serverless function (still free tier) that proxies the API call and hides the key.

## Frontend Spec

### Page Structure

1. **Header** - "Franciscan Woods HOA" + "Project Tracker" subtitle. Dark green background (#2C3E2D), sticky at top.

2. **Summary chips** - Row of pills showing count by status. e.g., "2 In Progress", "1 Planning", "1 Complete". Auto-generated from data.

3. **Filter buttons** - "All Projects" plus one button per status. Clicking filters the list. Active filter is highlighted green.

4. **Project cards** - One card per project row from the sheet. Each card has:

   **Always visible (collapsed):**
   - Project name (large, serif font)
   - Lead name, Units affected, Target date
   - Status badge (color-coded pill)
   - Urgency badge (color-coded small label)
   - "Next Action" bar at bottom showing: what needs to happen, who owns it, when it's due

   **Visible when expanded (click to toggle):**
   - Full description
   - Category and project type
   - Fund source
   - Vendors and bids
   - Selected vendor
   - Budget breakdown (budgeted / approved / actual)
   - Notes

### Sort Order
- Active projects sorted by urgency (Emergency > High > Medium > Low)
- Completed projects always at the bottom

### Color Scheme
- Status colors:
  - In Progress: orange (#D4740E)
  - Planning: purple (#7D6BA6)
  - Collecting Bids: blue (#2874A6)
  - Scheduled: teal (#2E86AB)
  - Complete: green (#3D6B4F)
- Urgency colors:
  - Emergency: red background
  - High: orange background
  - Medium: blue background
  - Low: gray background

### Typography
- Headings: Source Serif 4 (Google Fonts)
- Body: DM Sans (Google Fonts)
- Large, readable text throughout -- board members are older

### Mobile Responsive
- Cards stack full-width on mobile
- Meta info wraps naturally
- Touch-friendly tap targets

### Loading State
- Show a simple "Loading projects..." message while fetching from Google Sheets
- Show an error message if the sheet can't be reached

### Auto-refresh
- Fetch fresh data from the sheet every 5 minutes while the page is open
- No manual refresh needed

## File Structure

```
/
  index.html      (the entire app -- single file, HTML + CSS + JS)
  _headers         (Netlify headers file if needed)
```

Keep it as a single HTML file. No build tools, no npm, no framework. Plain vanilla JS. This makes it dead simple to update and deploy.

## Deployment

### Netlify Setup
1. Create a new site on Netlify
2. Drag and drop the folder containing index.html
3. Go to Site Settings > Access Control > Password Protection
4. Set a site-wide password
5. Share the URL + password with board members

### Updating the Password
When a board member leaves:
1. Log into Netlify dashboard
2. Site Settings > Access Control
3. Change the password
4. Text the new password to current board members

### Updating the Site
If the code itself needs changes:
1. Edit index.html locally
2. Drag and drop to Netlify again (it overwrites)

The project DATA is updated by editing the Google Sheet -- no redeployment needed for that.

## Configuration

At the top of the JavaScript in index.html, include two clearly labeled constants:

```javascript
// ============================================
// CONFIGURATION - Update these values
// ============================================
const SHEET_ID = 'your-google-sheet-id-here';
const API_KEY = 'your-google-api-key-here';
const SHEET_NAME = 'Projects';
const REFRESH_INTERVAL_MS = 300000; // 5 minutes
```

## What NOT to Build

- No edit functionality in the web app (editing happens in Google Sheets)
- No user authentication (Netlify password handles this)
- No backend server
- No database
- No notification system
- No file upload
- No comment system
- No history/changelog

## Testing Checklist

- [ ] Loads data from Google Sheet correctly
- [ ] All 5 sample projects display
- [ ] Cards expand/collapse on click
- [ ] Filter buttons work
- [ ] Status and urgency colors are correct
- [ ] Budget numbers format as currency
- [ ] Works on mobile (iPhone Safari, Android Chrome)
- [ ] Shows loading state while fetching
- [ ] Shows error state if sheet is unreachable
- [ ] Auto-refreshes after 5 minutes
- [ ] Empty columns in sheet don't break the display
- [ ] New rows added to sheet appear on next refresh
