# Landing Style Guide

## 1) Visual Direction

- Bright, airy canvas with soft gradients and large blur orbs.
- Strong contrast between text and background for readability.
- Clean, premium feel with rounded surfaces and subtle borders.

## 2) Background System

- Use a full-page wrapper that renders:
  - A layered gradient backdrop.
  - Two large blur orbs (one warm, one cool/green) positioned diagonally.
  - A subtle noise/overlay if needed for depth.
- Keep background layers behind content with `z-index` separation.

## 3) Layout

- Global structure: `PageShell` wraps all pages.
- Content container:
  - Max width for reading content (`max-w-4xl` for legal pages).
  - Centered container with consistent horizontal padding.

## 4) Header

- Sticky top bar:
  - `bg-background/80` + `backdrop-blur-md`.
  - Bottom border using `border-border/60`.
- Light, compact height; aligns brand link and toggles.

## 5) Typography

- Primary font: Wanted Sans Variable (already in globals).
- Headings:
  - Bold, tight tracking, large scale for hero or legal titles.
- Body:
  - Comfortable size with `text-foreground/80` for secondary copy.
- Prose blocks:
  - Use `prose` class with custom font family overrides.

## 6) Cards

- Surface:
  - `bg-card` with `border-border`.
  - Soft shadow and rounded corners.
- Use consistent padding and vertical rhythm.

## 7) Buttons

- Primary CTA:
  - High-contrast fill, strong hover state.
- Secondary:
  - Outline or muted background for less prominent actions.

## 8) Forms

- Inputs:
  - `bg-background`, `border-border`, and visible focus ring.
- Spacing:
  - Consistent vertical gaps (`space-y-*`).

## 9) Footer

- Simple, low-contrast band:
  - `border-t border-border`.
  - `bg-muted/20` with muted text.

## 10) Page-Specific Notes

- Privacy/Terms:
  - Structured sections with left border accents on headings.
  - Use `prose` and `space-y-*` for readability.
- Admin pages:
  - Same background wrapper; cards centered.

## 11) Design System

```css
@import url("https://cdn.jsdelivr.net/gh/wanteddev/wanted-sans@v1.0.3/packages/wanted-sans/fonts/webfonts/variable/split/WantedSansVariable.min.css");

@import "tailwindcss";
@import "tw-animate-css";
@import "shadcn/tailwind.css";

@custom-variant dark (&:is(.dark *));

@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --font-sans: var(--font-geist-sans);
  --font-mono: var(--font-geist-mono);
  --color-sidebar-ring: var(--sidebar-ring);
  --color-sidebar-border: var(--sidebar-border);
  --color-sidebar-accent-foreground: var(--sidebar-accent-foreground);
  --color-sidebar-accent: var(--sidebar-accent);
  --color-sidebar-primary-foreground: var(--sidebar-primary-foreground);
  --color-sidebar-primary: var(--sidebar-primary);
  --color-sidebar-foreground: var(--sidebar-foreground);
  --color-sidebar: var(--sidebar);
  --color-chart-5: var(--chart-5);
  --color-chart-4: var(--chart-4);
  --color-chart-3: var(--chart-3);
  --color-chart-2: var(--chart-2);
  --color-chart-1: var(--chart-1);
  --color-ring: var(--ring);
  --color-input: var(--input);
  --color-border: var(--border);
  --color-destructive: var(--destructive);
  --color-accent-foreground: var(--accent-foreground);
  --color-accent: var(--accent);
  --color-muted-foreground: var(--muted-foreground);
  --color-muted: var(--muted);
  --color-secondary-foreground: var(--secondary-foreground);
  --color-secondary: var(--secondary);
  --color-primary-foreground: var(--primary-foreground);
  --color-primary: var(--primary);
  --color-popover-foreground: var(--popover-foreground);
  --color-popover: var(--popover);
  --color-card-foreground: var(--card-foreground);
  --color-card: var(--card);
  --radius-sm: calc(var(--radius) - 4px);
  --radius-md: calc(var(--radius) - 2px);
  --radius-lg: var(--radius);
  --radius-xl: calc(var(--radius) + 4px);
  --radius-2xl: calc(var(--radius) + 8px);
  --radius-3xl: calc(var(--radius) + 12px);
  --radius-4xl: calc(var(--radius) + 16px);
}

:root {
  --font-serif:
    "Wanted Sans Variable", "Wanted Sans", -apple-system, BlinkMacSystemFont,
    system-ui, "Segoe UI", "Apple SD Gothic Neo", "Noto Sans KR",
    "Malgun Gothic", "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol",
    sans-serif;
  --radius: 0.625rem;
  --background: oklch(1 0 0); /* background: white */
  --foreground: oklch(0.145 0 0); /* textPrimary: black87 (approx) */
  --card: oklch(1 0 0); /* surface: white */
  --card-foreground: oklch(0.145 0 0);
  --popover: oklch(1 0 0);
  --popover-foreground: oklch(0.145 0 0);
  --primary: oklch(
    0.89 0.17 142
  ); /* primary: Color.fromARGB(255, 96, 255, 90) */
  --primary-foreground: oklch(0.205 0 0); /* Dark text on bright green */
  --secondary: oklch(0.97 0 0);
  --secondary-foreground: oklch(0.205 0 0);
  --muted: oklch(0.97 0 0);
  --muted-foreground: oklch(0.556 0 0);
  --accent: oklch(0.97 0 0);
  --accent-foreground: oklch(0.205 0 0);
  --destructive: oklch(0.577 0.245 27.325);
  --border: oklch(0.922 0 0);
  --input: oklch(0.922 0 0);
  --ring: oklch(0.89 0.17 142);
  --chart-1: oklch(0.89 0.17 142);
  --chart-2: oklch(0.6 0.118 184.704);
  --chart-3: oklch(0.398 0.07 227.392);
  --chart-4: oklch(0.828 0.189 84.429);
  --chart-5: oklch(0.769 0.188 70.08);
  --sidebar: oklch(0.985 0 0);
  --sidebar-foreground: oklch(0.145 0 0);
  --sidebar-primary: oklch(0.89 0.17 142);
  --sidebar-primary-foreground: oklch(0.985 0 0);
  --sidebar-accent: oklch(0.97 0 0);
  --sidebar-accent-foreground: oklch(0.205 0 0);
  --sidebar-border: oklch(0.922 0 0);
  --sidebar-ring: oklch(0.89 0.17 142);
}

.dark {
  --background: oklch(0.145 0 0); /* backgroundDark: Color(0xFF121212) */
  --foreground: oklch(1 0 0); /* textPrimaryDark: white */
  --card: oklch(0.205 0 0); /* surfaceDark: Color(0xFF1E1E1E) */
  --card-foreground: oklch(1 0 0);
  --popover: oklch(0.205 0 0);
  --popover-foreground: oklch(1 0 0);
  --primary: oklch(
    0.63 0.15 142
  ); /* primaryDark: Color.fromARGB(255, 76, 175, 80) */
  --primary-foreground: oklch(1 0 0);
  --secondary: oklch(0.269 0 0);
  --secondary-foreground: oklch(0.985 0 0);
  --muted: oklch(0.269 0 0);
  --muted-foreground: oklch(0.708 0 0);
  --accent: oklch(0.269 0 0);
  --accent-foreground: oklch(0.985 0 0);
  --destructive: oklch(0.704 0.191 22.216);
  --border: oklch(1 0 0 / 10%);
  --input: oklch(1 0 0 / 15%);
  --ring: oklch(0.63 0.15 142);
  --chart-1: oklch(0.63 0.15 142);
  --chart-2: oklch(0.696 0.17 162.48);
  --chart-3: oklch(0.769 0.188 70.08);
  --chart-4: oklch(0.627 0.265 303.9);
  --chart-5: oklch(0.645 0.246 16.439);
  --sidebar: oklch(0.205 0 0);
  --sidebar-foreground: oklch(0.985 0 0);
  --sidebar-primary: oklch(0.63 0.15 142);
  --sidebar-primary-foreground: oklch(1 0 0);
  --sidebar-accent: oklch(0.269 0 0);
  --sidebar-accent-foreground: oklch(0.985 0 0);
  --sidebar-border: oklch(1 0 0 / 10%);
  --sidebar-ring: oklch(0.63 0.15 142);
}

@layer base {
  * {
    @apply border-border outline-ring/50;
  }
  body {
    @apply bg-background text-foreground;
    font-family:
      "Wanted Sans Variable",
      "Wanted Sans",
      -apple-system,
      BlinkMacSystemFont,
      system-ui,
      "Segoe UI",
      "Apple SD Gothic Neo",
      "Noto Sans KR",
      "Malgun Gothic",
      "Apple Color Emoji",
      "Segoe UI Emoji",
      "Segoe UI Symbol",
      sans-serif;
  }
  .prose {
    font-family:
      "Wanted Sans Variable",
      "Wanted Sans",
      -apple-system,
      BlinkMacSystemFont,
      system-ui,
      "Segoe UI",
      "Apple SD Gothic Neo",
      "Noto Sans KR",
      "Malgun Gothic",
      "Apple Color Emoji",
      "Segoe UI Emoji",
      "Segoe UI Symbol",
      sans-serif;
  }
  .prose code {
    font-family:
      ui-monospace, SFMono-Regular, "SF Mono", Menlo, Consolas,
      "Liberation Mono", monospace;
  }
  .prose-headings\:font-serif
    :where(h1, h2, h3, h4, h5, h6, th):not(
      :where([class~="not-prose"], [class~="not-prose"] *)
    ) {
    font-family:
      "Wanted Sans Variable", "Wanted Sans", ui-serif, Georgia, Cambria,
      "Times New Roman", Times, serif;
  }
}
```
