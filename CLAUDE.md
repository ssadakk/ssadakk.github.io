# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an AstroPaper blog site built with Astro framework. The blog is deployed to GitHub Pages at https://ssadakk.github.io/ and focuses on technical writing about Android, Kotlin, and software engineering.

## Development Commands

```bash
# Install dependencies (using npm)
npm install

# Start development server at http://localhost:4321
npm run dev

# Build production site with Pagefind search and checks
npm run build

# Preview production build locally
npm run preview

# Format code with Prettier
npm run format

# Check code formatting
npm run format:check

# Run ESLint
npm run lint

# Generate TypeScript types for Astro modules
npm run sync
```

## Project Architecture

### Core Technologies
- **Framework**: Astro (v5.12.0) - Static site generator
- **Styling**: TailwindCSS v4 with Vite integration
- **Search**: Pagefind - Static search that builds during production build
- **Markdown Processing**: Remark plugins for TOC and collapsible sections
- **Code Highlighting**: Shiki with custom transformers for syntax highlighting
- **Type Safety**: TypeScript with strict mode enabled

### Content Structure
- **Blog Posts**: Located in `src/data/blog/` as Markdown files with frontmatter
- **Post Schema**: Each post requires:
  - `title`: Post title
  - `pubDatetime`: Publication date
  - `description`: Post summary
  - `tags`: Array of tags (defaults to ["others"])
  - Optional: `modDatetime`, `featured`, `draft`, `ogImage`, `canonicalURL`
- **Draft Posts**: Set `draft: true` in frontmatter to hide posts in production
- **Scheduled Posts**: Posts with future `pubDatetime` are hidden until that time passes

### Key Configuration Files
- **`src/config.ts`**: Site-wide configuration including metadata, pagination, and features
- **`src/content.config.ts`**: Content collection schema definition for blog posts
- **`astro.config.ts`**: Astro configuration with integrations and build settings

### Important Features
- **Dynamic OG Images**: Auto-generated social media images for each blog post
- **Light/Dark Mode**: Toggle between themes with persistence
- **Fuzzy Search**: Pagefind provides fast client-side search
- **SEO Optimized**: Sitemap, RSS feed, and meta tags configured
- **Google AdSense**: Integration ready via environment variable
- **Responsive Design**: Mobile-first with accessibility features

### Path Aliases
- `@/*` maps to `./src/*` for cleaner imports

### Build Process
The build command performs these steps in order:
1. `astro check` - Type checks the codebase
2. `astro build` - Builds static site to `./dist/`
3. `pagefind --site dist` - Indexes content for search
4. Copies pagefind assets to public directory

### Environment Variables
Optional environment variables can be set in `.env`:
- `PUBLIC_GOOGLE_SITE_VERIFICATION`: Google Search Console verification
- `PUBLIC_GOOGLE_ADSENSE_ID`: Google AdSense publisher ID

## Code Quality Standards

### Linting Rules
- ESLint with TypeScript and Astro plugins
- No console.log statements allowed (enforced by ESLint)
- Prettier formatting with specific rules:
  - 2 space indentation
  - 80 character line limit
  - No semicolons in arrow functions
  - ES5 trailing commas

### File Organization
- Components use `.astro` extension for Astro components
- Utilities are TypeScript modules in `src/utils/`
- All icons are SVG files in `src/assets/icons/`
- Static assets go in `public/` directory

## Common Development Tasks

### Adding a New Blog Post
1. Create a new `.md` file in `src/data/blog/` with format `YYYY-MM-DD-slug.md`
2. Add required frontmatter (title, pubDatetime, description)
3. Write content using Markdown with optional Remark plugins
4. Test locally with `npm run dev`

### Modifying Site Configuration
Edit `src/config.ts` to change:
- Site metadata (title, author, description)
- Pagination settings (postPerIndex, postPerPage)
- Feature toggles (showArchives, showBackButton, editPost)
- Timezone and language settings

### Working with Components
- Astro components are in `src/components/`
- Layout components are in `src/layouts/`
- Pages are in `src/pages/` and follow file-based routing