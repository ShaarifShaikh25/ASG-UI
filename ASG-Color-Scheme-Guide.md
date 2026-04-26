# ASG App Color Scheme Guide

Date: 26 April 2026
Project: ASG App (Apex Startup Group)

## 1) Brand Direction
ASG visual identity is built around a warm Orange to Red gradient, supported by deep neutrals and clean grays.

Core mood:
- Energetic
- Bold
- Modern
- High contrast

## 2) Primary Brand Colors

| Token | Value | Role |
|---|---|---|
| --orange-start | #FF8A00 | Gradient start (warm orange) |
| --orange-end | #FF3D00 | Gradient end (deep orange-red) |
| --orange-gradient | linear-gradient(180deg, #FF8A00 0%, #FF3D00 100%) | Main brand gradient |
| --orange-gradient-soft | linear-gradient(180deg, rgba(255,138,0,0.2) 0%, rgba(255,61,0,0.2) 100%) | Soft tinted surfaces |

## 3) Extended Orange Palette

| Token | Value | Role |
|---|---|---|
| --orange | #FF5A14 | Fallback accent orange |
| --orange-mid | #FF7A2B | Mid accent / text badges |
| --orange-dark | #E24200 | Dark accent for stronger emphasis |
| --orange-light | #FFF0E5 | Light orange tint for hover/background chips |

## 4) Neutrals and Base UI Colors

| Token | Value | Role |
|---|---|---|
| --black | #0D0D0D | Primary dark surfaces and text |
| --black2 | #1A1A1A | Secondary dark hover/sections |
| --gray-1 | #F7F7F7 | App background |
| --gray-2 | #EDEDED | Borders and secondary surfaces |
| --gray-3 | #D4D4D4 | Input borders and muted separators |
| --gray-4 | #9A9A9A | Muted text |
| --white | #FFFFFF | Cards, foreground text on dark/gradient |

## 5) Support and Status Colors

| Token/Value | Role |
|---|---|
| #E8F5E9 | Success badge background |
| #2E7D32 | Success badge text |

## 6) Transparency Layers Used

- rgba(255,138,0,0.2) and rgba(255,61,0,0.2): Soft brand overlays and chips
- rgba(255,61,0,0.35): Branded soft border
- rgba(255,255,255,0.1 to 0.9): Text/surface overlays on dark backgrounds
- rgba(0,0,0,0.08 to 0.4): Shadows and overlays

## 7) Where the Brand Gradient Is Applied

Main gradient is used in:
- Primary buttons
- Brand logo accent dot/text accent
- User avatars and repo tags
- Active tab/section text highlights
- Key headings and numeric highlights (gradient-clipped text)
- Active filter pills
- Progress and active state accents

Soft brand gradient is used in:
- Selected/active cards
- Soft badges (AI, College Tool, Registration chip)
- Hero accent overlays

## 8) Contrast and Accessibility Notes

- White text is used on dark and gradient backgrounds for readability.
- Body copy remains neutral/dark for long reading comfort.
- Accent gradient should be used for emphasis, not for large body paragraphs.
- On very light backgrounds, prefer dark text with orange border/accent instead of full gradient text blocks.

## 9) Quick Usage Rules for Team

1. For CTA buttons, always use --orange-gradient.
2. For subtle highlighted backgrounds, use --orange-gradient-soft.
3. For borders in active state, use --orange-end.
4. For neutral UI structure, use gray tokens only.
5. Do not introduce new random orange hex values outside token system.

## 10) Approved Color Set (Final)

Primary:
- #FF8A00
- #FF3D00
- #FF5A14
- #FF7A2B
- #E24200
- #FFF0E5

Neutrals:
- #0D0D0D
- #1A1A1A
- #F7F7F7
- #EDEDED
- #D4D4D4
- #9A9A9A
- #FFFFFF

Support:
- #E8F5E9
- #2E7D32

This is the official ASG App color system currently implemented in the app.
