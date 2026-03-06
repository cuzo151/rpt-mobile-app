# RPT Mobile App — MVP Design

**Date:** 2026-03-05
**Client:** Rohnert Park Transmission & Auto Repair
**Website:** rohnertparktransmission.com
**Stack:** HTML/CSS/JS PWA → Supabase backend → n8n automation → Claude AI

---

## Overview

Mobile app for RPT's clients to track repair progress, book appointments, and monitor vehicle maintenance. Shop staff updates repair status with a single tap; clients see live updates in-app.

## Brand Identity

Match the RPT website: clean, professional, trust-focused.

### Light Theme (Default)
- Background: `#FFFFFF`
- Sections: `#F9FAFB` (gray-50)
- Cards: white, `#E5E7EB` borders, subtle shadows
- Text primary: `#111827` (gray-900)
- Text secondary: `#6B7280` (gray-500)
- Accent: `#dc2626` (rohnert-red)
- Secondary accent: `#f59e0b` (rohnert-yellow)
- Navigation: white with subtle border

### Dark Theme (Toggle)
- Background: `#000000`
- Cards: `#0A0A0A`, `rgba(220,38,38,0.15)` borders
- Text: `#FFFFFF` / `rgba(255,255,255,0.4)`
- Same red/yellow accents

### Shared
- Typography: Orbitron (display) + Rajdhani (body)
- Toggle: sun/moon icon in header
- Persists via localStorage, respects prefers-color-scheme on first visit

---

## Architecture

### Frontend (PWA)
- Evolve existing `home-screen.html` into multi-screen PWA
- `manifest.json` + service worker for install
- Light/dark theme via CSS custom properties + `data-theme` attribute
- 5 screens: Home, Tracker, Bookings, Maintenance, Profile

### Backend (Supabase)
- **Auth:** Phone number + SMS OTP
- **Database tables:**
  - `customers` — name, phone, email
  - `vehicles` — make, model, year, VIN, mileage, customer_id
  - `appointments` — service_type, date, time, status, vehicle_id
  - `repair_orders` — order_number, status, tech_id, timeline_steps[], estimate_items[], vehicle_id
  - `maintenance_schedule` — service_type, last_performed, interval_miles, interval_months, vehicle_id
  - `service_history` — completed services with dates and costs
  - `notifications` — type, message, read, customer_id, created_at
- **Real-time:** Subscriptions on repair_orders and notifications tables

### Admin Panel
- Separate page for shop staff
- Staff login (email/password)
- Repair order management: tap status buttons to advance through timeline
- View incoming appointment requests, approve/reschedule
- Single tap triggers: real-time client update + push notification + AI-generated message

### Notifications (In-App, not SMS)
- PWA push notifications (browser-native)
- Supabase real-time for live tracker updates when app is open
- Notification bell with unread count in header
- Notification feed screen

### Automation (n8n)
- **Webhook trigger:** Supabase fires on repair_orders status change
- **Switch node:** Routes by status value
- **Claude AI node:** Generates contextual notification message
- **Supabase node:** Writes to notifications table (triggers push + real-time)
- **Daily schedule:** Check maintenance_schedule, create reminder notifications for upcoming services

### AI Layer (Claude)
- Smart estimate generation from service type + vehicle
- Contextual notification messages (not generic)
- Intake categorization from client issue descriptions

---

## Staff Workflow (6 taps per repair)

```
Received → Diagnosed → Parts Ordered → In Progress → Quality Check → Ready for Pickup
```

Each tap triggers:
1. Repair timeline updates in real-time for client
2. Push notification with AI-written contextual message
3. Notification appears in client's notification feed
4. On completion: maintenance schedule auto-recalculates

---

## Build Phases

### Phase 1 — Frontend PWA + Theme Toggle
- Restyle all 5 screens for light theme (default)
- Dark theme as toggle option
- Add manifest.json + service worker
- Notification bell + notification feed screen

### Phase 2 — Supabase Backend + Auth
- Create database schema
- Phone number OTP auth
- Connect frontend to Supabase (CRUD, real-time)

### Phase 3 — Admin Panel
- Staff login
- Repair order status management (tap to advance)
- Appointment approval queue

### Phase 4 — n8n Automation
- Status change → notification workflow
- Daily maintenance reminder workflow

### Phase 5 — Claude AI Integration
- Smart estimate generation
- Contextual notification messages
- Intake categorization

---

## External Services Required

| Service | Purpose | Cost | MVP Required? |
|---------|---------|------|---------------|
| Supabase | Database, auth, real-time | Free tier | Yes |
| n8n | Automation workflows | Free self-hosted / $20/mo | Phase 4 |
| Anthropic API | Claude AI features | Pay per use | Phase 5 |
| Vercel/Netlify | Hosting | Free tier | Yes |
| Twilio | SMS backup (optional) | ~$0.008/SMS | No — nice-to-have |
