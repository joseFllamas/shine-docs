# Shine — Design Specification for Google Stitch

## 1. Project Overview

**Shine** is a mobile-first app for people with dyslexia. It helps users improve reading and writing through structured exercises (called *activities*), organized into *sessions*. Progress is tracked visually so users and therapists can see improvement over time.

**Target users**: Children and adults with dyslexia. Many have difficulty reading dense or cluttered interfaces. The UI must be calm, spacious, and confidence-building — never stressful or noisy.

**Platforms**: iOS, Android, and Web (same codebase via Expo / React Native Web).

---

## 2. Design Principles

1. **Calm and encouraging** — warm colors, generous whitespace, no visual clutter.
2. **Accessible by default** — WCAG AA minimum. High contrast, large touch targets (≥48px), large readable text (minimum 18sp body).
3. **Dyslexia-friendly typography** — sans-serif font (OpenDyslexic or similar rounded sans-serif like Nunito). No justified text. Line height ≥ 1.6. Letter spacing slightly open.
4. **One thing at a time** — each screen has a single primary action. No competing CTAs.
5. **Progress is visible** — users should always feel they are moving forward.
6. **Positive reinforcement** — celebrate completions with subtle animation (confetti, color pulse), never punish mistakes.

---

## 3. Color Palette

| Role | Color | Notes |
|---|---|---|
| Primary | Warm teal `#3DBFA0` | Main CTA buttons, active states |
| Primary dark | `#2A8C74` | Pressed states, text on light bg |
| Accent | Warm amber `#F5A623` | Progress bars, highlights |
| Background | Off-white `#F7F5F2` | Main screen bg — never pure white |
| Surface | White `#FFFFFF` | Cards, modals |
| Text primary | Dark charcoal `#1E1E2E` | Main content |
| Text secondary | Medium gray `#6B7280` | Labels, captions |
| Success | Soft green `#4CAF7D` | Completed states |
| Error | Warm red `#E05A5A` | Errors only — use sparingly |

Avoid pure black and pure white. Use the off-white background to reduce visual fatigue.

---

## 4. Typography

- **Font family**: Nunito (or OpenDyslexic as user preference)
- **Heading 1**: 28sp, Bold, charcoal, letter-spacing 0.3px
- **Heading 2**: 22sp, SemiBold, charcoal
- **Body**: 18sp, Regular, line-height 1.7, charcoal
- **Label**: 14sp, Medium, secondary gray
- **Activity text** (the words/phrases shown during exercises): 24–32sp, Bold or SemiBold, high contrast, centered

Never use font sizes below 14sp anywhere in the app.

---

## 5. Screen Map

```
Login
  └── OAuth2 login (opens browser)

Dashboard (home)
  ├── User greeting + weekly summary stats
  ├── List of available sessions (cards)
  └── Link to Statistics

Session Detail
  ├── Session title + description
  ├── List of activities (with completion status)
  └── Start / Continue button

Activity Player (one of three types, see §6)
  ├── Intro phase
  ├── Playing phase
  └── Completed phase (motivation message + confetti)

Statistics
  ├── Overview cards (total activities, total time, this week)
  ├── Activity evolution chart (line chart per activity)
  ├── Standard time comparison (bar chart)
  └── Session comparison table
```

---

## 6. Screens — Detailed Descriptions

### 6.1 Login Screen

**Layout**: Full-screen, centered, single CTA.

- Large Shine logo / wordmark at center top
- Short motivational tagline below: *"Let's practice together"*
- Large primary button: **"Sign in"** (opens OAuth2 browser flow)
- The screen is completely distraction-free — no nav, no footer

---

### 6.2 Dashboard

**Layout**: Scrollable vertical list, safe area padding.

**Top section — Greeting card**:
- "Good morning, [Name]" in Heading 1
- Small weekly stats inline: `5 activities this week · 24 min total`
- Subtle gradient card background (light teal tint)

**Section: "Your sessions"**:
- Heading 2: "Sessions"
- Each session is a card:
  - Session title (Heading 2)
  - Short description (Body, 2 lines max, truncated)
  - Progress bar showing activities completed (e.g. `7 / 10`)
  - Tapping the card → Session Detail
- Cards have 16px border radius, soft shadow, white background on off-white page bg

**Bottom nav** (sticky):
- Home icon (active on Dashboard)
- Statistics icon

---

### 6.3 Session Detail

**Layout**: Scrollable, back button top-left.

**Header**:
- Session title (Heading 1)
- Description text (Body)
- SessionProgressBar component: `████████░░ 8/10 activities completed`

**Activity list**:
- Each activity is a row item:
  - Activity icon (by type: text, audio, image)
  - Activity title
  - Status badge: `Completed` (green) or `Pending` (gray)
  - If completed: show best time as a small label
- Tapping an activity → Activity Player

**Sticky bottom CTA**:
- Primary button: **"Continue"** (starts first uncompleted activity) or **"Start"** if none completed yet

---

### 6.4 Activity Player — Intro Phase

Every activity starts here before the exercise begins.

**Layout**: Full-screen, vertically centered, no nav.

**Elements**:
- Activity title (Heading 1, centered)
- Difficulty badge: pill chip (`Beginner` / `Intermediate` / `Advanced`)
- Explanation message box: card with soft teal border, icon, and explanation text
- Custom explanation text if present (Body, below the card)
- Large primary button at bottom: **"Start"**

When the user taps Start → timer begins → Playing Phase.

---

### 6.5 Activity Player — Playing Phase (Vertical Text / Blink)

The main exercise. Shows words or phrases one at a time.

**Layout**: Full-screen, distraction-free. No nav. No header. Only the word and a subtle progress indicator.

**Elements**:
- Current word/phrase: very large centered text (28–36sp, Bold)
  - `blink` format: word auto-advances after N seconds (configurable). Subtle fade transition.
  - `vertical_list` format: user taps anywhere on screen to advance. Slide-up animation.
- Progress dots or counter at bottom (e.g. `3 / 12`)
- Tiny "Skip" text link at top-right (escape hatch, for accessibility)

**Visual design**: high contrast, word centered on off-white background. The word should feel like it "breathes" on the page. No other UI elements competing for attention.

---

### 6.6 Activity Player — Playing Phase (Audio Reading)

The user reads the text out loud. The app measures reading time.

**Layout**: Full-screen, distraction-free.

**Elements**:
- Full text displayed (multiple lines, large font, well spaced)
- Large circular microphone button at center-bottom:
  - Idle: outlined circle with mic icon, label "Tap to start reading"
  - Recording: pulsing red circle, label "Reading…", elapsed timer below
  - Done: success green circle, checkmark icon
- When the user taps the mic: recording starts + timer starts
- "Done reading" button appears after 2s of recording
- No other distractions

---

### 6.7 Activity Player — Completed Phase

Shown after every activity completes.

**Layout**: Full-screen celebration moment.

**Elements**:
- Confetti animation (subtle, 3 seconds)
- Large checkmark icon in success green, animated scale-in
- "Activity complete!" (Heading 1, centered)
- Time taken: e.g. `Your time: 42 seconds`
- Motivational message card (randomly selected, warm and encouraging, e.g. *"You're doing great! Keep going!"*)
- Two buttons:
  - **"Next activity"** (primary) — if more activities remain in session
  - **"Back to session"** (secondary, outlined)

---

### 6.8 Statistics Screen

**Layout**: Scrollable, header with back button and title "My Progress".

**Section 1 — Overview cards** (horizontal scroll row):
- Card: "Total activities" — large number + icon
- Card: "This week" — number + sparkline
- Card: "Total time" — formatted duration

**Section 2 — Activity Evolution**:
- Title: "How am I improving?"
- Dropdown to select which activity to view
- Line chart: X = attempt number, Y = time in seconds. Line trends down = improving.
- Below chart: collapsible accessible data table (same data in text form, for screen readers)

**Section 3 — Vs. Standard**:
- Title: "Compared to average"
- Horizontal bar chart: my best time vs. standard time for the selected activity
- If no standard time is configured, show: *"No reference configured for this activity"*

**Section 4 — Session History**:
- Title: "Session history"
- Table:
  | Session | Date | Activities | Total time |
  |---|---|---|---|
  - Rows sorted newest first

**Accessibility notes for all charts**:
- High contrast colors (no light gray lines on white)
- Data labels directly on chart (not only in legend)
- Every chart has an accessible data table below it (collapsible)
- No information conveyed by color alone (use icons or labels too)

---

## 7. Shared Components

### Button

- **Primary**: filled teal background, white text, 48px height, 12px border-radius, full-width on mobile
- **Secondary**: white background, teal border, teal text
- **Destructive**: white background, red border, red text (used for logout only)
- All buttons: 48px minimum tap target height

### Card

- White background, 16px border-radius, shadow `0 2px 8px rgba(0,0,0,0.08)`
- 16px internal padding
- Subtle hover/press state: slight scale-down (0.98) + deeper shadow

### Badge / Chip

- Activity difficulty: pill shape, 8px border-radius
  - Beginner: light green bg, dark green text
  - Intermediate: light amber bg, dark amber text
  - Advanced: light purple bg, dark purple text

### Progress Bar

- Background: light gray `#E5E7EB`
- Fill: warm amber `#F5A623`
- Height: 12px, fully rounded ends
- Always includes a text label: `7 / 10 activities`

### MotivationMessage Card

- Soft teal left border (4px)
- Light teal tint background
- Icon (star or heart) + message text in Body size
- Used in: completed phase, activity intro

---

## 8. Navigation Structure

```
Bottom Navigation Bar (only on authenticated screens):
  [Home / Dashboard]    [Statistics]
```

- No hamburger menus
- Sessions and activities use a simple push navigation (back arrow top-left)
- Activity player hides all navigation during the exercise (full-screen focus mode)

### Web adaptation

The design is **mobile-first**. On web (Expo Web / browser):

- Constrain the entire layout to **max-width 480px**, centered on screen, with a neutral `#EEECE9` outer background filling the sides
- The bottom navigation bar becomes a **top header bar** with the same two items (Home, Statistics) plus the app logo/wordmark on the left
- All other components, spacing, and font sizes remain identical — no responsive breakpoints needed beyond this single constraint
- The activity player full-screen focus mode still covers the full 480px column (not the browser viewport)

---

## 9. Accessibility Requirements

- All interactive elements: minimum 48×48px tap target
- Color contrast: WCAG AA minimum for all text
- Screen reader labels on all icon-only elements
- Font size: user can scale via system settings (no fixed pixel sizes that break scaling)
- No auto-playing sound or animations without user action
- Confetti animation: respects `prefers-reduced-motion`
- Data tables available as alternative to every chart

---

## 10. Empty States

- **No sessions yet**: illustration + "No sessions available yet. Check back soon."
- **No statistics yet**: illustration + "Complete your first activity to see your progress here."
- **Session fully completed**: celebration banner + "You've completed all activities in this session!"

Empty states should always include an illustration (simple, friendly, dyslexia-themed — books, pencils, lightbulbs) and a short encouraging message. Never a blank screen.

---

## 11. Micro-interactions

- Activity word transitions: `fade` (blink format) or `slide-up` (vertical_list format), 200ms ease
- Button press: scale 0.97, 100ms
- Card tap: scale 0.98, 100ms
- Completed checkmark: scale from 0 to 1.1 then settle to 1.0, 300ms spring
- Progress bar fill: animated from previous value on load, 500ms ease-out
- Confetti: canvas-based, 3 seconds, fades out gracefully
