# GitHub Copilot Instructions — domainforvijay.in

## Project Overview

This is **Vijay Bambhaniya's** personal website and blog, built with **Hugo** using the **PaperMod** theme. It is hosted on **GitHub Pages** with a custom domain `domainforvijay.in`.

- **Framework:** Hugo (static site generator)
- **Theme:** PaperMod
- **Domain:** https://domainforvijay.in/
- **Repository:** https://github.com/00imvj00/00imvj00.github.io
- **Google Analytics ID:** G-4TWMQ7ZZ4P

## Site Structure

```
content/
  about.md              — About page
  posts/                — Blog posts ("Thoughts" section)
    _index.md
  tenets/               — Opinionated engineering principles ("Tenets" section)
    _index.md
    anatomy-of-a-healthy-service.md
    writing-technical-documents.md
    reviewing-technical-documents.md
```

## Content Sections

| Section | Path | Menu Label | Purpose |
|---------|------|------------|---------|
| Home | `/` | Home | Landing page with intro |
| Posts | `/posts/` | Thoughts | Blog entries, reflections, ideas |
| Tenets | `/tenets/` | Tenets | Opinionated engineering principles |
| About | `/about/` | About | Resume / bio page |

## Creating New Content — Mandatory Rules

### Front Matter (YAML format, between `---`)

Every new content file **must** include all of the following front matter fields:

```yaml
---
title: "Descriptive, Keyword-Rich Title"
date: YYYY-MM-DD
description: "A compelling 150-160 character meta description that includes primary keywords and encourages clicks from search results."
tags: ["tag1", "tag2", "tag3"]
draft: false
weight: <number>  # Only for tenets — controls ordering
---
```

#### Field Requirements

- **title**: Concise, descriptive, includes primary keyword. Keep under 60 characters for optimal search display.
- **date**: ISO format (`YYYY-MM-DD`). Use the current date unless the user specifies otherwise.
- **description**: 150-160 characters. This becomes the `<meta name="description">` tag and the OpenGraph/Twitter card description. It must be unique per page, compelling, and keyword-rich. Never leave this empty.
- **tags**: 2-5 relevant tags as an array. Reuse existing tags when possible. Current tags in use:
  - `architecture`, `operations`, `services`
  - `technical-design`, `documentation`, `code-review`
  - `software engineering`, `technical leadership`
  - `about`
- **draft**: Set to `false` for published content. Use `true` only if the user explicitly says it's a draft.
- **weight**: Only for tenets. Increment from the last tenet's weight.

### Blog Post Template (for `content/posts/`)

```yaml
---
title: "Your SEO-Optimized Title Here"
date: 2026-02-15
description: "A 150-160 char description with keywords that appears in search results and social shares."
tags: ["relevant-tag-1", "relevant-tag-2"]
draft: false
---

## Opening Hook

Start with a strong opening that establishes the problem or context.

---

## Main Content

Structure with clear H2/H3 headings for readability and SEO.

---

## Key Takeaways

Summarize actionable insights.
```

### Tenet Template (for `content/tenets/`)

```yaml
---
title: "#N — Tenet Title"
date: 2026-02-15
description: "A 150-160 char description of the engineering principle."
tags: ["relevant-tag-1", "relevant-tag-2"]
draft: false
weight: N
---

## The Core Belief

A strong opinionated statement about the principle.

---

## Content sections...
```

## SEO Checklist — Apply to Every New Page

When creating or editing any content file, verify:

1. **`title`** — Under 60 chars, includes primary keyword
2. **`description`** — 150-160 chars, unique, compelling, keyword-rich
3. **`tags`** — 2-5 relevant tags (reuse existing ones when applicable)
4. **Headings** — Use a logical H2 → H3 hierarchy (never skip levels). Include keywords naturally in headings.
5. **Internal links** — Link to other pages on the site where relevant (e.g., `[healthy service](/tenets/anatomy-of-a-healthy-service/)`)
6. **URL slug** — The filename becomes the URL slug. Use lowercase, hyphen-separated, keyword-rich names (e.g., `building-resilient-microservices.md`)
7. **Image alt text** — If images are added, always include descriptive alt text
8. **First paragraph** — Should contain the primary keyword naturally within the first 100 words

## SEO Infrastructure (Already Configured)

These are handled automatically by the theme + config. Do NOT remove or modify:

- `env = "production"` in `hugo.toml` — Enables OpenGraph, Twitter Cards, Schema JSON-LD, robots indexing
- `enableRobotsTXT = true` — Generates robots.txt with sitemap reference
- Canonical URLs — Auto-generated from `baseURL`
- Sitemap — Auto-generated at `/sitemap.xml`
- Google Analytics — Tracking via `G-4TWMQ7ZZ4P`
- OpenGraph & Twitter meta tags — Auto-generated from `title`, `description`, and `images`
- Schema.org JSON-LD — Person schema on homepage, Breadcrumb schema on inner pages
- RSS feed — Available at `/index.xml`
- Keywords meta tag — Site-level keywords set in `hugo.toml`

## File Naming Conventions

- **Posts:** `content/posts/my-post-title.md` — lowercase, hyphenated
- **Tenets:** `content/tenets/descriptive-name.md` — lowercase, hyphenated
- **Images:** Place in `static/images/` — use descriptive filenames

## Writing Style

- Vijay writes in a direct, opinionated, practitioner-first voice
- Content is grounded in real-world experience, not theory
- Uses analogies and strong assertions followed by evidence
- Markdown formatting: liberal use of bold, code blocks, tables, and diagrams
- Sections are separated with `---` horizontal rules

## Common Tasks

### Adding a new blog post
1. Create file: `content/posts/<slug>.md`
2. Add full front matter (title, date, description, tags, draft: false)
3. Write content with proper heading hierarchy
4. Add internal links to related tenets/posts where relevant
5. Verify: `hugo --minify` builds without errors

### Adding a new tenet
1. Check the latest weight in existing tenets
2. Create file: `content/tenets/<slug>.md`
3. Use tenet front matter template with incremented weight
4. Title format: `"#N — Tenet Name"`
5. Start with `## The Core Belief` section
6. Verify: `hugo --minify` builds without errors

### Building and previewing
- **Dev server:** `hugo server -D` (includes drafts)
- **Production build:** `hugo --minify`
- **Output directory:** `public/`

## Things to Never Do

- Never create a content file without `description` in front matter
- Never use duplicate descriptions across pages
- Never skip the `tags` field
- Never use `draft: true` unless explicitly asked
- Never modify files inside `themes/PaperMod/` — use `layouts/` overrides instead
- Never hardcode the domain — always let Hugo generate URLs from `baseURL`
