# Product

<!-- impeccable:product-schema 1 -->

## Platform

web

## Stack

Astro (delegated: content-focused, fast static output, excellent for documentation/library sites; supports Islands for interactive search/filter)

## Users

Developers building apps with OpenCode who need to discover, evaluate, and install agent skills to extend their workflow. They are typically in a coding session, have a specific problem to solve, and want a trusted skill without trial-and-error.

## Product Purpose

A curated skill library for OpenCode that makes it trivial to find the right skill for a task, understand what it does, and install it with confidence. Success means a developer searches "how do I animate a modal", finds the `animate` skill, reads its purpose, and installs it — all in under 30 seconds.

## Positioning

The only skill registry that combines: (1) human-curated quality bar (no auto-generated listings), (2) task-oriented discovery ("what skill do I need for X?"), and (3) one-command install that wires the skill into the user's opencode config. Neighboring products (npm, GitHub search) cannot truthfully claim all three.

## Operating Context

- Used inside a developer's coding workflow (IDE, terminal, browser)
- Skills are installed per-project or globally via opencode CLI
- Each skill is a self-contained folder with SKILL.md, agents, and optional scripts
- Developers may browse online or offline; the site must work as a static reference

## Capabilities and Constraints

- Skill catalog with search, filtering by category/tags, and detail pages
- One-click install command (copies to `.opencode/skills/` or project `.agents/skills/`)
- Offline-first: all skill metadata baked into static build
- No backend, no auth, no user accounts
- Skills are versioned with the repo; updates via git pull
- WCAG AA compliance required
- Must render correctly on mobile and desktop

## Brand Commitments

None. Greenfield project — no existing name, logo, voice, colors, or identity constraints.

## Evidence on Hand

No real content, testimonials, case studies, press, or assets exist yet. Future work must not fabricate usage numbers, adoption metrics, or fictional endorsements.

## Product Principles

1. **Task-first, not catalog-first** — Organize by what the developer is trying to do, not by skill name.
2. **Trust over volume** — Fewer, vetted skills beat a noisy marketplace.
3. **Install in one step** — Zero-config adoption; the install command is the primary CTA.
4. **Static by default** — No runtime dependencies; works offline, deploys anywhere.
5. **Accessible by construction** — Semantic HTML, keyboard navigation, and contrast are not afterthoughts.

## Accessibility & Inclusion

WCAG 2.1 AA compliance required. All interactive elements keyboard-operable, sufficient color contrast (4.5:1 text, 3:1 UI), semantic heading structure, descriptive link text, and alt text for any imagery. Reduced motion respected via `prefers-reduced-motion`.