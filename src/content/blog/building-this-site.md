---
title: "Building gustaviversen.dev with Astro 6 and Tailwind v4"
description: "How I built this portfolio using Astro 6, Tailwind CSS v4 with CSS-first configuration, and deployed it to Cloudflare Pages."
publishDate: 2026-04-03
tags: ["Astro", "Tailwind CSS", "Web Development"]
draft: false
---

This site runs on Astro 6 with static output, styled with Tailwind CSS v4 using the new CSS-first @theme configuration. Here is a look at the technical decisions behind it.

## Why Astro

Content-driven sites benefit from zero-JS-by-default. Astro ships no client-side JavaScript unless you explicitly opt in with `client:` directives. For a portfolio and blog that is mostly static content, this is ideal -- pages load fast, and the build output is plain HTML and CSS.

## Design system

The design tokens are defined in a single `global.css` file using Tailwind v4's `@theme` block. This replaces the old `tailwind.config.js` approach entirely. Every color, font, and spacing value flows from CSS custom properties.

The site shares structural patterns with the HAVEN Intelligence consulting site (glass header, card-glow effects, noise texture overlay) but uses a warm gold/bronze palette instead of cyan, and Inter Variable bold for headings instead of Instrument Serif.

## Content collections

Blog posts and projects are both managed through Astro's Content Layer API with the `glob()` loader. Schemas are defined with Zod in `src/content.config.ts`, which provides type safety across the entire build pipeline. Dates use `z.coerce.date()` to handle YAML string-to-Date conversion.

## Deployment

The site builds to static HTML and deploys to Cloudflare Pages. No server-side rendering, no edge functions -- just `npm run build` producing a `dist/` directory that Cloudflare serves globally.
