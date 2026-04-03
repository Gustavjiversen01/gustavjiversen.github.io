---
title: "gustaviversen.dev"
description: "This personal portfolio and blog. Dark-theme developer site with a warm gold accent, built on the same Astro + Tailwind foundation as the HAVEN consulting site."
repo: "https://github.com/gustaviversen/personal-website"
demo: "https://gustaviversen.dev"
tags: ["Astro", "Tailwind CSS", "Portfolio", "Cloudflare Pages"]
featured: true
publishDate: 2026-04-03
---

My personal developer portfolio and blog. Shares the same structural patterns as the HAVEN Intelligence site but establishes its own visual identity through a desaturated gold palette, warmer charcoal backgrounds, and sans-serif headings.

## Design decisions

- Gold/bronze accent palette instead of HAVEN's cyan, creating warmth without the "construction site" problem that saturated amber causes on dark backgrounds
- Inter Variable for headings (bold), reserving Instrument Serif only for the hero name and blockquotes
- Slightly warmer background tones to support the warm accent colors
- Terminal motif recurring across hero and 404 page

## Technical stack

- Astro 6 with Content Layer API (glob loader, content.config.ts)
- Tailwind CSS v4 with @tailwindcss/typography for markdown prose
- RSS feed via @astrojs/rss
- Static output deployed to Cloudflare Pages
