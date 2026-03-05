# RayDBG Blog v3

A minimalist, terminal-inspired blog built with [Astro](https://astro.build/).

## Features

- **Terminal aesthetic** - Monospace fonts, dark theme, command-line inspired UI
- **Markdown blog** - Write posts in Markdown with frontmatter
- **Type-safe content** - Astro Content Collections with Zod validation
- **Zero JavaScript** - Static site generation for optimal performance
- **Minimal design** - Clean, distraction-free reading experience

## Quick Start

```bash
# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview
```

## Writing Blog Posts

Create new blog posts in `src/content/blog/`:

```markdown
---
title: "My First Post"
description: "A brief description"
pubDate: 2024-01-15
tags: ["astro", "blog"]
---

Your content here...
```

### Frontmatter Fields

| Field | Type | Description |
|-------|------|-------------|
| `title` | string | Post title (required) |
| `description` | string | Post description (optional) |
| `pubDate` | date | Publication date (required) |
| `updatedDate` | date | Last updated date (optional) |
| `tags` | string[] | Post tags (optional) |
| `draft` | boolean | Mark as draft (optional, default: false) |

## Project Structure

```
src/
├── content/
│   ├── blog/          # Blog posts (Markdown)
│   └── config.ts      # Content collection schema
├── layouts/           # Astro layouts
├── pages/             # Route pages
└── styles/            # Global CSS
```

## Tech Stack

- [Astro](https://astro.build/) - Web framework
- [TypeScript](https://www.typescriptlang.org/) - Type safety
- [Zod](https://zod.dev/) - Schema validation

## License

MIT