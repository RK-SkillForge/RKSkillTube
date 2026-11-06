---
name: Obsidian Kinetic
colors:
  surface: '#0f131c'
  surface-dim: '#0f131c'
  surface-bright: '#353942'
  surface-container-lowest: '#0a0e16'
  surface-container-low: '#181c24'
  surface-container: '#1c2028'
  surface-container-high: '#262a33'
  surface-container-highest: '#31353e'
  on-surface: '#dfe2ee'
  on-surface-variant: '#c7c4d8'
  inverse-surface: '#dfe2ee'
  inverse-on-surface: '#2c3039'
  outline: '#918fa1'
  outline-variant: '#464555'
  surface-tint: '#c3c0ff'
  primary: '#c3c0ff'
  on-primary: '#1d00a5'
  primary-container: '#4f46e5'
  on-primary-container: '#dad7ff'
  inverse-primary: '#4d44e3'
  secondary: '#adc6ff'
  on-secondary: '#002e6a'
  secondary-container: '#0566d9'
  on-secondary-container: '#e6ecff'
  tertiary: '#d0bcff'
  on-tertiary: '#3c0091'
  tertiary-container: '#6f3dd9'
  on-tertiary-container: '#e3d5ff'
  error: '#ffb4ab'
  on-error: '#690005'
  error-container: '#93000a'
  on-error-container: '#ffdad6'
  primary-fixed: '#e2dfff'
  primary-fixed-dim: '#c3c0ff'
  on-primary-fixed: '#0f0069'
  on-primary-fixed-variant: '#3323cc'
  secondary-fixed: '#d8e2ff'
  secondary-fixed-dim: '#adc6ff'
  on-secondary-fixed: '#001a42'
  on-secondary-fixed-variant: '#004395'
  tertiary-fixed: '#e9ddff'
  tertiary-fixed-dim: '#d0bcff'
  on-tertiary-fixed: '#23005c'
  on-tertiary-fixed-variant: '#5516be'
  background: '#0f131c'
  on-background: '#dfe2ee'
  surface-variant: '#31353e'
typography:
  display-xl:
    fontFamily: Geist
    fontSize: 48px
    fontWeight: '700'
    lineHeight: 56px
    letterSpacing: -0.03em
  display-xl-mobile:
    fontFamily: Geist
    fontSize: 32px
    fontWeight: '700'
    lineHeight: 40px
    letterSpacing: -0.02em
  headline-lg:
    fontFamily: Geist
    fontSize: 32px
    fontWeight: '600'
    lineHeight: 40px
    letterSpacing: -0.02em
  headline-lg-mobile:
    fontFamily: Geist
    fontSize: 24px
    fontWeight: '600'
    lineHeight: 32px
    letterSpacing: -0.02em
  headline-md:
    fontFamily: Geist
    fontSize: 24px
    fontWeight: '600'
    lineHeight: 32px
    letterSpacing: -0.015em
  headline-sm:
    fontFamily: Geist
    fontSize: 18px
    fontWeight: '600'
    lineHeight: 26px
    letterSpacing: -0.01em
  body-lg:
    fontFamily: Inter
    fontSize: 16px
    fontWeight: '400'
    lineHeight: 24px
    letterSpacing: -0.01em
  body-md:
    fontFamily: Inter
    fontSize: 14px
    fontWeight: '400'
    lineHeight: 20px
    letterSpacing: -0.005em
  body-sm:
    fontFamily: Inter
    fontSize: 12px
    fontWeight: '400'
    lineHeight: 18px
    letterSpacing: '0'
  label-md:
    fontFamily: JetBrains Mono
    fontSize: 12px
    fontWeight: '500'
    lineHeight: 16px
    letterSpacing: 0.02em
  label-sm:
    fontFamily: JetBrains Mono
    fontSize: 10px
    fontWeight: '500'
    lineHeight: 14px
    letterSpacing: 0.04em
  code-inline:
    fontFamily: JetBrains Mono
    fontSize: 13px
    fontWeight: '400'
    lineHeight: 18px
    letterSpacing: '0'
rounded:
  sm: 0.125rem
  DEFAULT: 0.25rem
  md: 0.375rem
  lg: 0.5rem
  xl: 0.75rem
  full: 9999px
spacing:
  gutter: 1rem
  gutter-desktop: 1.5rem
  margin: 1rem
  margin-tablet: 1.5rem
  margin-desktop: 2rem
  space-xs: 0.25rem
  space-sm: 0.5rem
  space-md: 1rem
  space-lg: 1.5rem
  space-xl: 2.5rem
---

## Brand & Style

The design system is engineered for a high-performance tech education and streaming ecosystem. The brand identity balances the density, utility, and precision of developer tooling with the fluid, immersive qualities of modern streaming interfaces.

The aesthetic fuses **Linear-style utilitarian minimalism** with **deep dark-mode glassmorphism** and razor-sharp accents:
- **Tone & Mood:** Hyper-focused, technical, cinematic, and authoritative.
- **Target Audience:** Software engineers, system architects, and technical learners who expect information density, zero latency, and distraction-free visual environments.
- **Stylistic Signatures:** Translucent charcoal surfaces over deep obsidian canvases, hairline translucent borders (`1px solid rgba(255, 255, 255, 0.08)`), micro-glow volumetric highlights, high-contrast code-level typography, and tactical neon accent hits.

## Colors

The palette is tuned specifically for deep OLED/dark displays, prioritizing contrast ratios and reducing visual fatigue during long video and coding sessions.

### Functional Roles
- **Base Canvas (`#0B0F17`):** Deep obsidian, serving as the foundational viewport backdrop.
- **Surface Elevation 1 (`#111827`):** Dark slate for card bodies, video sidebars, and structural modules.
- **Surface Elevation 2 (`#1F2937`):** Elevated menus, modal containers, and hovering controls.
- **Primary / Brand Action (`#4F46E5` - Electric Indigo):** Primary buttons, active track indications, key focus rings, and highlighted navigation states.
- **Secondary / Telemetry (`#3B82F6` - Hyper Blue):** Interactive timestamps, timeline scrubbers, link highlights, and streaming metric counters.
- **Tertiary / Intelligence (`#8B5CF6` - Violet):** AI-generated code summaries, automated video chaptering, and intelligent quiz modules.
- **Success / Completion (`#10B981` - Emerald Green):** Course completion badges, progress meters, validated code test-cases, and uptime status.
- **Subtle Glass Borders (`rgba(255, 255, 255, 0.08)`):** Hairline edge-definitions that create structural depth without heavy opacity.

## Typography

The typographic architecture prioritizes rapid scanning, dense tabular data, and readable video metadata.

- **Display & Headlines (Geist):** Tight kerning, modern geometric neutral curves, and commanding weight without decorative flourishes.
- **Body & Continuous Reading (Inter):** Highly legible, tall x-height, and neutral structure for video descriptions, chat logs, and long-form transcripts.
- **Micro-labels, Telemetry & Badges (JetBrains Mono):** Monospaced precision for video duration stamps, bitrates, AI tokens, tags, and progress indicators.

## Layout & Spacing

The layout model uses a fluid 12-column responsive grid built for media streaming dashboards and technical consumption:

- **Desktop (1280px+):** 12-column layout with fixed-width collapsible navigation rail (64px collapsed, 240px expanded), an auto-expanding main theatre, and dynamic secondary sidebars (video playlist or live terminal/chat).
- **Tablet (768px - 1279px):** 8-column layout. The playlist collapses into overlay drawers or stacks beneath the video canvas.
- **Mobile (< 768px):** 4-column layout. Sticky 16:9 media player pinned to viewport top, with scrollable tabbed panels below.
- **Density:** Spacing tokens reflect high utility. Padding within components stays compact (`space-xs` to `space-md`), preserving vertical view real estate for video and curriculum trees.

## Elevation & Depth

Visual hierarchy does not rely on opaque drop shadows; instead, it utilizes a combination of **tonal layering**, **hairline borders**, and **subtle directional radiance**:

- **Level 0 (Canvas):** Pure `#0B0F17`.
- **Level 1 (Panels & Feed Cards):** `#111827` with `border: 1px solid rgba(255, 255, 255, 0.08)`.
- **Level 2 (Popovers, Dropdowns, Hovered Video Cards):** `#1F2937` with `border: 1px solid rgba(255, 255, 255, 0.14)` and a subtle ambient occlusion `box-shadow: 0 12px 32px -8px rgba(0, 0, 0, 0.6)`.
- **Level 3 (Modals & Overlays):** Semi-translucent `#111827` with `backdrop-filter: blur(16px)` and `border: 1px solid rgba(255, 255, 255, 0.18)`.
- **Accent Glows:** Hovering over interactive media cards or activating AI features casts an ambient light burst: `0 0 24px -4px rgba(79, 70, 229, 0.25)` or `0 0 24px -4px rgba(139, 92, 246, 0.25)`.

## Shapes

The design system embraces a **Soft (Level 1)** geometric standard to retain an intentional, software-instrument appearance:

- **Base Radius (0.25rem / 4px):** Micro-badges, inline code tags, video timestamps, progress pill ticks, and checkboxes.
- **Container Radius (0.5rem / 8px):** Video thumbnails, code blocks, input containers, buttons, and popovers.
- **Modal / Card Radius (0.75rem / 12px):** Video player boundaries, floating modal cards, and drawer interfaces.
- **Pill (Fully rounded):** Audio waveforms, filter chips, and live status badges.

## Components

### Buttons
- **Primary:** Background `#4F46E5`, text `#FFFFFF`, font `Inter` 14px weight 500. Subtle inner top-highlight `inset 0 1px 0 rgba(255, 255, 255, 0.2)`. On hover, background shifts to `#4338CA` with an ambient glow (`box-shadow: 0 0 16px rgba(79, 70, 229, 0.4)`).
- **Secondary:** Surface `#111827`, border `1px solid rgba(255, 255, 255, 0.08)`, text `#E5E7EB`. Hover: border brightens to `rgba(255, 255, 255, 0.2)`, background to `#1F2937`.
- **AI Action:** Gradient border or surface tint using `#8B5CF6` with violet particle accent icon.

### Chips & Filter Tags
- Height: 28px. Font: `Inter` 12px weight 500.
- Inactive: `#111827` surface, hairline border `rgba(255, 255, 255, 0.08)`, text `#9CA3AF`.
- Active: Background `rgba(79, 70, 229, 0.15)`, border `1px solid #4F46E5`, text `#FFFFFF`.

### Video Stream & Lesson Cards
- Surface `#111827` with `border: 1px solid rgba(255, 255, 255, 0.08)`.
- 16:9 aspect-ratio media canvas with overlaid bottom-right badge in `JetBrains Mono` (`background: rgba(0, 0, 0, 0.8)`, `padding: 2px 6px`).
- Hover: 1px border shifts to `rgba(79, 70, 229, 0.5)` with `-translate-y-1` transition (150ms ease-out).

### Input Fields
- Dark slate surface `#0F172A`, border `1px solid rgba(255, 255, 255, 0.1)`.
- Text `#F9FAFB`, placeholder `#6B7280`.
- Focus state: border `#3B82F6` accompanied by an outer shadow ring `0 0 0 2px rgba(59, 130, 246, 0.2)`.

### Code Blocks & Terminal Snippets
- Surface `#090D14`, hairline border `1px solid rgba(255, 255, 255, 0.06)`.
- Header bar containing file name in `JetBrains Mono` 11px, tab indicators, and a one-click copy trigger with Emerald Green check feedback.

### Progress & Scrub Bars
- Base track: `rgba(255, 255, 255, 0.1)`.
- Buffer track: `rgba(255, 255, 255, 0.2)`.
- Progress indicator: `#4F46E5` transitioning into `#3B82F6` at scrubber tip, with completed course tracks marked in `#10B981`.