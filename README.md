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

### Create a New Post

1. Create a new Markdown file in `src/content/blog/`
2. Use the naming convention: `YYYY-MM-DD-slug.md`
3. Add the required frontmatter at the top

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

### Markdown Syntax Examples

```markdown
# Heading 1
## Heading 2
### Heading 3

**Bold text** and *italic text*

- Bullet list item 1
- Bullet list item 2
  - Nested item

1. Numbered list item 1
2. Numbered list item 2

[Link text](https://example.com)

![Image alt text](/images/path/to/image.jpg)

> Blockquote

---

Inline `code` and code blocks:

```
function hello() {
  console.log("Hello, World!");
}
```

| Table | Header |
|-------|--------|
| Cell 1 | Cell 2 |
```

### Tips

- Use `<!-- more -->` to create a preview excerpt on the blog listing page
- Put images in `public/images/` directory
- Set `draft: true` to hide a post during development

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