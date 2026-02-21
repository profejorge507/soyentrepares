# AI Coding Agent Instructions for WebAstro

## Project Overview
**WebAstro** is a portfolio/blog site built with [Astro](https://astro.build) featuring a blog with content collections, responsive gallery with interactive modals, and dark mode support. Language: Spanish (es).

## Architecture & Key Components

### Content Collections (Type-Safe Blog)
- **File**: [src/content/config.ts](src/content/config.ts) defines the `blog` collection schema
- **Pattern**: Frontmatter uses Zod validation with required fields: `title`, `description`, `pubDate`, optional: `updatedDate`, `heroImage`, `tags`
- **Usage**: Blog posts in [src/content/blog/](src/content/blog/) are queried via `getCollection('blog')` sorted by `pubDate` descending
- **Example**: See [src/pages/blog.astro](src/pages/blog.astro#L6-L11) for collection fetching pattern

### Layout System
- **Base Layout**: [src/layouts/BlogPost.astro](src/layouts/BlogPost.astro) wraps individual blog post pages using `getStaticPaths()`
- **Page Patterns**: Each `.astro` page imports layout components (Header, Footer, BaseHead) that establish consistent structure
- **Type Safety**: Use `type Props = CollectionEntry<'blog'>['data']` for type-checked frontmatter

### Component Patterns
1. **Modal Component** ([src/components/Modal.astro](src/components/Modal.astro)): Reusable dialog with XSS protection, ARIA attributes, detail lists, and CSS-based animations (no JS runtime required)
2. **Theme Toggle** ([src/components/ThemeToggle.astro](src/components/ThemeToggle.astro)): Dark mode via `html.dark` class with CSS variables
3. **Header Navigation** ([src/components/Header.astro](src/components/Header.astro#L26-L35)): Client-side menu toggle for mobile responsiveness

### Styling Conventions
- **CSS Variables**: Use `var(--surface)`, `var(--gray-dark)`, `var(--black)` for theming
- **Dark Mode**: Prefix dark styles with `:global(html.dark)` selector
- **Grid Layouts**: Blog cards use CSS Grid with `repeat(auto-fill, minmax(300px, 1fr))`
- **Backdrop Effects**: Modals use `backdrop-filter: blur()` with `-webkit-` prefixes for compatibility
- **File**: [src/styles/global.css](src/styles/global.css) defines color palette and base styles

## Build & Deployment Commands
```bash
npm run dev          # Start dev server at localhost:4321
npm run build        # Build to ./dist/ directory
npm run preview      # Preview production build locally
npm run astro -- --help  # Access Astro CLI
```

## Project-Specific Patterns

### Site Configuration
- **File**: [src/consts.ts](src/consts.ts) - Import `SITE_TITLE`, `SITE_DESCRIPTION` globally; update here for site-wide changes
- **Config**: [astro.config.mjs](astro.config.mjs) uses MDX and Sitemap integrations; `site:` property needed for RSS/SEO

### Blog Post Creation
1. Create `.md` or `.mdx` file in [src/content/blog/](src/content/blog/)
2. Use required frontmatter: title, description, pubDate (YYYY-MM-DD format)
3. Optional: heroImage (path to public asset), tags (array), updatedDate

### Tag-Based Filtering
- [src/pages/[tag].astro](src/pages/[tag].astro) dynamically generates tag pages; filtering logic uses `post.data.tags?.includes(tag)`

### Responsive Design Strategy
- Fixed header at top (`position: fixed; z-index: 1000`) requires page padding
- Mobile-first: Define base styles, then use `media (min-width: 640px)` for larger breakpoints
- Hamburger menu visibility controlled by CSS with `.menu-toggle.active` state

## Integration & Data Flow

### Markdown → HTML Pipeline
- Astro processes [src/content/blog/](src/content/blog/) markdown files at build time
- `getCollection()` returns typed collection entries; call `.render()` to get `<Content />` component
- [src/pages/blog/[...slug].astro](src/pages/blog/[...slug].astro#L6-L11) example: dynamic routes via `getStaticPaths()`

### Asset Handling
- Static assets in [public/](public/) served directly (no Astro processing)
- Use `import` for images in `.astro` to enable optimization: `import { Image } from 'astro:assets'`

## Common Tasks

**Add a new blog post**: Create [src/content/blog/my-post.md](src/content/blog/) with frontmatter + markdown

**Update navigation**: Modify [src/components/Header.astro](src/components/Header.astro#L13-L18) `HeaderLink` components

**Change site title/description**: Update [src/consts.ts](src/consts.ts) exports

**Customize modal styling**: Edit `:root` variables or `.modal-overlay`/`.modal-content` styles in [src/components/Modal.astro](src/components/Modal.astro#L42+)

**Add dark mode support**: Use `:global(html.dark)` selector in component `<style>` blocks and CSS variables

## Dependencies
- **astro** (^5.17.1): Core framework
- **@astrojs/mdx** (^4.3.13): MDX support for blog posts
- **@astrojs/rss**, **@astrojs/sitemap**: RSS feed and sitemap generation
- **sharp** (^0.34.3): Image optimization

## Avoid Common Pitfalls
- ⚠️ Blog frontmatter date must be coerced to Date type; use `pubDate: 2025-02-01` format
- ⚠️ Modal IDs must be unique; pass `id={uniqueValue}` prop for aria-labelledby targeting
- ⚠️ Client-side interactivity: Use `<script>` tags in `.astro` files (inline scripts run only on page load)
- ⚠️ Styling scoped within components unless explicitly using `:global()` for shared styles
