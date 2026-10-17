> **Prompt:**
> "create a design system and core screens for multi-pages web-based developer portfolio for desktop that serves as a multi-pages routing experience.

> **Global Layout:**
> A fixed left sidebar navigation with a circular avatar, vertical navigation links of beautiful (Home, About, Skills, Projects, Experience, Education, Recommendations, Contact), and a Download CV button at the bottom. The right side contains the main scrollable content.
> The app has both light and dark modes.

> **Include the following screens:**
>
> - **Hero:** Large 'Mojahed Mohammed' header, 'FRONTEND ENGINEER' overline, and intro text and environment the reflect front-end developer work with Next and React .
> - **About:** A portrait photo (tech image) framed with decorative minimal corner brackets, placed next to a biography.
> - **Skills:** A responsive grid of small, rounded square cards, each displaying a tech logo and name.
> - **Projects:** Large feature cards with project preview images on top, titles, descriptions, and pill-shaped tech tags.
> - **Experience:** A vertical timeline connected by a thin line on the left, featuring dark cards with company logos, job titles, and dates.
> - **Recommendations:** A grid of testimonial cards featuring a quote icon, paragraph text, and user avatars with job titles at the bottom.
> - **Contact:** A modern input form with dark background fields on the left, and contact information block-cards on the right. Add a floating AI chat widget interface in the bottom right corner with an open state showing a welcome message.
>
> **Target users:** Tech recruiters and engineering managers looking for a clean, high-performance, and visually striking interface."

Searched for files: _wattvision_
Viewed design-md-sistema-de-monitoreo-el-ctrico-wattvision-DESIGN.md:1-35

Here is the regenerated prompt aligned strictly with your **WattVision Design System (`DESIGN.md`)**.

Since Google Stitch already has `WattVision DESIGN.md` loaded in context, this prompt directly references its color tokens (`#121212`, `#1E1E1E`, `#00E5FF`, `#FF453A`, `#32D74B`, `#2C2C2E`), card specifications (`16px` radius, `20px` padding), typography rules (Inter + JetBrains Mono for metrics), and alert styling (`#3A1C1C`).

---

# 🎨 Google Stitch Prompt: Portfolio Admin Routes & 404 Page (WattVision Design System)

> **Copy and paste the prompt below into Google Stitch:**

```markdown
# UI/UX Generation Task: Portfolio Admin Suite & 404 Error Page

Strictly adhere to the WattVision Design System (`DESIGN.md`) currently loaded in context for all screens, tokens, colors, typography, card structures, and table layouts.

---

## 1. Design System Enforcement Rules (WattVision Compliance)

- **Background**: `#121212` (Primary Dark).
- **Cards & Surfaces**: `#1E1E1E` background, `border-radius: 16px`, `padding: 20px`, subtle border `1px solid #2C2C2E`.
- **Primary Data Accent & Actions**: `#00E5FF` (Neon Cyan) for primary buttons, KPI metrics, active chart paths, and focused controls.
- **Secondary Status Accent**: `#32D74B` (Lime Green) for "Active/Published/Live" status tags and normal metric indicators.
- **Alert & Destructive Accent**: `#FF453A` (Red) for delete triggers, error states, and unread alerts.
- **Alert Containers**: Background `#3A1C1C`, text `#FF453A`, left border `4px solid #FF453A`.
- **Typography**:
  - Titles & Section Headers: **Inter SemiBold 24px**, `#FFFFFF`.
  - KPI Metrics & Numerical Counters: **JetBrains Mono / Fira Code Bold 32px**, `#00E5FF`.
  - Body & Form Text: **Inter Regular 14px**, `#FFFFFF` (Primary) / `#98989D` (Secondary labels).
- **Tables**: `border-bottom: 1px solid #2C2C2E` per row, row hover `background: #252525`.

---

## 2. Screen Generation Requirements

### Screen 1: Main Admin Dashboard (`/admin`)

- **Global Admin Layout Shell**: Fixed Left Sidebar Navigation (Side Navbar) featuring brand logo ("DEV // ADMIN"), vertical navigation links (Dashboard, Skills, Experience, Projects, Messages [with unread count badge], "← Live Portfolio"), and user avatar trigger. Main content area sits to the right of the Side Navbar with an Inter SemiBold 24px page title ("Control Panel / Dashboard").
- **Top Row (3 KPI Cards - 12-Column Grid Split)**:
  - 3 equal cards (`#1E1E1E`, `16px` radius, `1px solid #2C2C2E`, `20px` padding).
  - Metric 1: **Total Portfolio Views** — `32px` JetBrains Mono in `#00E5FF`, small trend tag in `#32D74B` ("+14% this month").
  - Metric 2: **Active Projects** — `32px` JetBrains Mono in `#00E5FF`, subtitle "Published live".
  - Metric 3: **Unread Messages** — `32px` JetBrains Mono in `#FF453A`, with subtle `#3A1C1C` alert glow.
- **Main Area (12-Column Grid)**:
  - **Left Area (8 Columns)**: "Content Mutation Log" — table showing recent edits (Project created, Skill updated, Experience added) with `#252525` row hover.
  - **Right Area (4 Columns)**: "Priority Messages Panel" — alert boxes formatted with background `#3A1C1C`, text `#FF453A`, and left border `4px solid #FF453A` for unread client messages.

---

### Screen 2: Resource Management Tables (`/admin/projects`, `/admin/skills`, `/admin/experience`)

- **Layout**: Full-width `#1E1E1E` surface table container with `16px` border-radius and `1px solid #2C2C2E` border.
- **Header Control Bar**:
  - Search input with `#121212` background and `#2C2C2E` border.
  - Action Button: "+ Add New Record" in `#00E5FF` (Neon Cyan) with dark text (`#121212`) Inter SemiBold.
- **Table Data Layout**:
  - Row dividers: `1px solid #2C2C2E`. Row hover background: `#252525`.
  - Columns: `ID (JetBrains Mono #98989D)` | `Title / Name` | `Category / Tech Stack` | `Status Tag` | `Created Date` | `Actions`
  - Status Tag Pill: `#32D74B` background tint for "Published/Active", `#98989D` for "Draft".
  - Action Triggers: Edit button in `#00E5FF` outline, Delete trigger in `#FF453A` red text.

---

### Screen 3: CRUD Modal Dialog / Form Drawer (Create & Edit Work / Skills)

- **Modal Surface**: `#1E1E1E` card with `16px` border-radius, `20px` padding, and `1px solid #2C2C2E` border.
- **Form Controls**:
  - Input Fields: `#121212` dark background, `#2C2C2E` border, `14px` Inter text in `#FFFFFF`, label in `#98989D`.
  - Image Dropzone (Vercel Blob asset upload): `#121212` background, dashed `#2C2C2E` border, upload icon in `#00E5FF`.
  - Tech Stack / Tags Input: Inter Regular 14px pills with `#2C2C2E` border and `#00E5FF` text.
- **Footer Buttons**:
  - Cancel: Ghost button with `#98989D` text.
  - Save / Update: Solid `#00E5FF` (Neon Cyan) button with `#121212` Inter SemiBold text.

---

### Screen 4: Messages Inbox (`/admin/messages`)

- **Master-Detail Layout**:
  - **Left Pane (4 Columns)**: Message list on `#1E1E1E` card. Unread messages styled with `#3A1C1C` alert background and `4px solid #FF453A` left border. Read messages styled with normal `#252525` hover state.
  - **Right Pane (8 Columns)**: Selected Message Details viewer.
    - Sender name in Inter SemiBold 24px `#FFFFFF`.
    - Sender email in JetBrains Mono 14px `#00E5FF`.
    - Message body in Inter Regular 14px `#FFFFFF`.
    - Actions: "Mark as Read" toggle in `#32D74B`, "Delete Message" button in `#FF453A` red outline.

---

### Screen 5: 404 Error Screen (`/404`)

- **Atmosphere**: Technical, dark `#121212` background with subtle `#2C2C2E` grid lines.
- **Hero Center Block**:
  - **Error Code**: `404` displayed in **JetBrains Mono Bold 64px** in `#00E5FF` (Neon Cyan).
  - **Title**: Inter SemiBold 24px `#FFFFFF` ("Signal Lost: Page Not Found").
  - **Body**: Inter Regular 14px `#98989D` ("The requested route does not exist or has been relocated in the system.").
- **Actions**:
  - Primary CTA Button: "← Return to Dashboard / Home" in `#00E5FF` background with `#121212` text.
  - Secondary Action Bar: Quick links in `#1E1E1E` cards with `1px solid #2C2C2E` ("Projects", "Experience", "Contact").
```

---

### Key Highlights of this Prompt:

1. **Direct System Integration**: Matches WattVision's exact color palette (`#121212`, `#1E1E1E`, `#00E5FF`, `#FF453A`, `#32D74B`, `#2C2C2E`).
2. **Typography Rules Enforced**: Uses **Inter** for titles/body and **JetBrains Mono** for numbers, KPIs, email addresses, and the `404` header.
3. **Alert Box Accuracy**: Employs WattVision's exact alert style (`#3A1C1C` background + `4px solid #FF453A` left border) for unread messages and dashboard notifications.
