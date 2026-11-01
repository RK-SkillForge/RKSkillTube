# RKSkillTube UI Design System & Prototypes

Welcome to the **UI** directory of RKSkillTube. This folder serves as the central design workspace and prototype library for the platform's user interface.

Here you will find visual specifications, interactive HTML/Tailwind prototypes, and design system tokens used to build the Next.js frontend.

---

## 📁 Directory Structure

```text
UI/
├── README.md               # Overview and UI design documentation (this file)
├── Home Page/              # Landing page and main feed designs
│   ├── DESIGN.md           # Design tokens, color palette, typography & specs
│   ├── code.html           # Standalone interactive HTML/Tailwind prototype
│   └── screen.png          # Visual design preview & screenshot
└── Admin page/             # Admin management & creator dashboard designs
    └── (In progress)       # Upcoming specs, prototypes, and screen mocks
```

---

## 🎨 Design Theme: *Obsidian Kinetic*

The UI is built around the **Obsidian Kinetic** design philosophy—tailored specifically for technical developers and learners who need high information density, deep dark-mode ergonomics, and distraction-free learning.

### 1. Color Palette

Optimized for OLED and low-light environments with high contrast and subtle neon/electric accents:

| Token / Role | Hex Code | Purpose & Usage |
| :--- | :--- | :--- |
| **Canvas Background** | `#0B0F17` / `#0F131C` | Deep obsidian base surface |
| **Surface (Container)** | `#111827` / `#1C2028` | Cards, sidebars, panels |
| **Surface Elevated** | `#1F2937` / `#262A33` | Dropdowns, dialogs, active menus |
| **Primary Brand** | `#4F46E5` / `#C3C0FF` | Primary action buttons, active navigation states |
| **Secondary (Telemetry)** | `#3B82F6` / `#ADC6FF` | Video scrubbers, timestamps, links |
| **Tertiary (AI & Magic)** | `#8B5CF6` / `#D0BCFF` | AI Study Tutor, smart roadmap generator |
| **Success** | `#10B981` | Completed course badges, verified progress |

### 2. Typography

We use three complementary typefaces:

- **Geist (Display & Headlines):** Modern, geometric, and clean for titles and section headings.
- **Inter (Body & UI text):** High legibility for long descriptions, navigation labels, and video titles.
- **JetBrains Mono (Badges & Code):** Monospaced precision for timestamps, tags, terminal blocks, and video metadata.

### 3. Surface & Depth

- **Hairline Glass Borders:** Minimal translucent borders (`border: 1px solid rgba(255, 255, 255, 0.08)`) instead of heavy opaque drop shadows.
- **Ambient Glows:** Soft colored radiance (`box-shadow: 0 0 24px -4px rgba(79, 70, 229, 0.25)`) on active AI elements and interactive cards.
- **Rounded Corners:** Consistent `0.5rem` (8px) for cards, `0.25rem` (4px) for badges, and full pills for search and filters.

---

## 🚀 How to View and Use These Designs

### 1. Previewing Standalone Prototypes
You can preview the interactive prototypes without running any backend or dev server:
1. Navigate into any page folder (e.g., `UI/Home Page/`).
2. Double-click or open `code.html` directly in your web browser.
3. You will see the responsive UI layout with live Tailwind styling and Material Symbols.

### 2. Inspecting Design Tokens
Each page folder contains a `DESIGN.md` file specifying:
- Exact color variables and Tailwind configurations.
- Spacing scales and font size hierarchies.
- Component breakdown (buttons, video cards, search bars, navigation rails).

### 3. Implementing in the Next.js Frontend
When converting these prototypes into production React components in `frontend/`:
- Map the classes in `code.html` to reusable components inside `frontend/src/components/`.
- Ensure theme tokens match the config in `frontend/tailwind.config.ts`.
- Follow accessibility guidelines (ARIA labels, keyboard navigation, readable contrast).

---

## 🛠 Adding a New Page Design

If you are designing a new screen (e.g., *Admin Page*, *Course Detail*, *Video Player*):

1. Create a dedicated folder under `UI/` (e.g., `UI/Admin page/`).
2. Add your reference files:
   - `code.html`: Standalone HTML prototype with Tailwind classes.
   - `DESIGN.md`: Specification detailing layout, components, and interactions.
   - `screen.png`: Screenshot or mockup for quick visual reference.
3. Update this `README.md` file to reference the new page in the directory structure.

---

*Need help or want to contribute? Check out the main [CONTRIBUTING.md](../CONTRIBUTING.md) guide!*
