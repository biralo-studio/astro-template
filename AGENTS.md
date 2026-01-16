# Agent Guidelines for Astro Projects

Astro 5 static site with Svelte 5 islands for interactivity. Deployed via Cloudflare with DecapCMS for content management.

## Tech Stack
- **Framework**: Astro 5
- **Interactive Components**: Svelte 5 (with runes)
- **Styling**: SCSS with @use syntax
- **Language**: TypeScript
- **Testing**: Vitest
- **Maps**: MapLibre GL (optional)
- **Deployment**: Cloudflare Pages
- **CMS**: DecapCMS

## Build Commands

- **Dev server**: `npm run dev` (localhost:4321)
- **Build**: `npm run build`
- **Preview**: `npm run preview`
- **Run tests**: `npm test`
- **Run tests once**: `npm run test:run`

## Testing Guidelines

### Unit Tests (Vitest)
- Tests located in `src/tests/`
- Run with `npm test` (watch mode) or `npm run test:run` (single run)
- Test utilities in `src/tests/lib/`

### Visual Testing with Screenshots
- Use headless Chrome for visual verification
- Run `npm run dev` locally, then capture screenshots:

```bash
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
    --headless \
    --screenshot=/tmp/screenshot.png \
    --window-size=1440,900 \
    http://localhost:4321/
```

- Test responsive layouts: mobile (375px), tablet (768px), desktop (1440px)
- Verify color accuracy, spacing, typography, and interactive states

## Project Structure

```
src/
├── content/              # Astro Content Collections
│   ├── config.ts         # Zod schemas for all collections
│   ├── posts/            # Example collection
│   └── [collection]/     # Other collections as needed
│
├── pages/                # Astro pages and API routes
│   ├── index.astro
│   ├── [collection]/
│   │   ├── index.astro           # Listing page
│   │   └── [...slug].astro       # Detail pages
│   └── api/                      # API endpoints (optional)
│       └── [collection].json.ts
│
├── layouts/
│   └── BaseLayout.astro
│
├── components/
│   ├── ui/               # Static Astro components
│   │   ├── Header.astro
│   │   ├── Footer.astro
│   │   └── cards/
│   │
│   └── interactive/      # Svelte 5 islands
│       └── *.svelte
│
├── lib/                  # TypeScript business logic
│   ├── types/            # TypeScript interfaces
│   ├── stores/           # Svelte 5 rune stores (*.svelte.ts)
│   ├── utils/            # Utility functions
│   └── api/              # API helpers
│
├── styles/
│   └── global/           # SCSS partials
│       ├── _variables.scss
│       ├── _breakpoints.scss
│       ├── _typography.scss
│       └── _base.scss
│
└── tests/                # Vitest tests
    └── lib/

public/
├── assets/               # Static images, fonts
│   ├── images/
│   └── fonts/
└── admin/                # DecapCMS
    └── config.yml
```

## Code Style Guidelines

### General
- No trailing whitespace
- Use UTF-8 encoding
- Follow DRY principles
- Use meaningful variable and function names
- Handle errors gracefully
- Browser support: Modern browsers with ES6+

### Astro Components (.astro)
- Use for static/server-rendered content
- Frontmatter script between `---` fences
- Access content collections via `getCollection()`
- Use `getStaticPaths()` for dynamic routes

```astro
---
import { getCollection } from 'astro:content';
import BaseLayout from '../layouts/BaseLayout.astro';

const items = await getCollection('posts');
---
<BaseLayout title="Posts">
  {items.map(item => <div>{item.data.title}</div>)}
</BaseLayout>
```

### Svelte 5 Components (.svelte)
- Use for interactive "islands" only
- **Use Svelte 5 runes**: `$state`, `$derived`, `$effect`
- **NO legacy syntax**: Don't use `$:`, `export let`, or stores from `svelte/store`
- Place in `src/components/interactive/`
- Hydrate with `client:load` or `client:visible`

```svelte
<script lang="ts">
  let count = $state(0);
  let doubled = $derived(count * 2);

  function increment() {
    count++;
  }
</script>

<button onclick={increment}>
  Count: {count}, Doubled: {doubled}
</button>
```

### Svelte 5 Stores (*.svelte.ts)
- Place in `src/lib/stores/`
- Use runes for reactive state

```typescript
// filterStore.svelte.ts
export function createFilterStore() {
  let filters = $state({ city: [], category: [] });

  return {
    get filters() { return filters; },
    setFilter(type: string, values: string[]) {
      filters[type] = values;
    }
  };
}
```

### TypeScript
- Strict mode enabled in tsconfig.json
- Interfaces in `src/lib/types/`
- Use Zod for content collection schemas
- Type API responses properly

### SCSS/CSS
- **Use `@use` syntax** (NOT `@import` - deprecated)
- Variables auto-imported via Vite config (no manual import needed)
- Class names use kebab-case: `.detail-card`, `#main-nav`
- Use mixins for responsive breakpoints:
  - `@include mobile` (max-width: 768px)
  - `@include tablet` (768px - 1100px)
  - `@include tablet-or-smaller` (max-width: 1100px)
  - `@include desktop` (min-width: 1101px)
- Use CSS Grid and Flexbox for layouts
- Mobile-first responsive design
- Use `rem` units for spacing and typography

### Content Collections
- Defined in `src/content/config.ts` with Zod schemas
- Access with `getCollection('collection-name')`
- Markdown files with YAML frontmatter

**Example schema (config.ts):**
```typescript
import { defineCollection, z } from 'astro:content';

const posts = defineCollection({
  type: 'content',
  schema: z.object({
    title: z.string(),
    date: z.date(),
    description: z.string().optional(),
    image: z.string().optional(),
  }),
});

export const collections = { posts };
```

**Example frontmatter:**
```yaml
---
title: "Article Title"
date: 2024-01-15
description: "Brief description"
image: "/assets/images/featured.jpg"
---
Content here...
```

### API Endpoints
- Located in `src/pages/api/*.json.ts`
- Use `APIRoute` type from Astro
- Support filtering via URL search params

```typescript
import type { APIRoute } from 'astro';
import { getCollection } from 'astro:content';

export const GET: APIRoute = async ({ url }) => {
  const items = await getCollection('posts');
  const category = url.searchParams.get('category');

  const filtered = category
    ? items.filter(i => i.data.category === category)
    : items;

  return new Response(JSON.stringify(filtered));
};
```

## Performance Optimization
- Astro outputs zero JS by default for static pages
- Use `client:load` sparingly - only for interactive components
- Use `client:visible` for below-fold interactivity
- Optimize images using Cloudflare Images
- Use lazy loading for images below the fold

## SEO Guidelines
- Include descriptive meta titles (50-60 characters)
- Write compelling meta descriptions (150-160 characters)
- Use proper heading hierarchy (H1 → H2 → H3)
- Include structured data (JSON-LD) where appropriate
- Use descriptive URLs with keywords

## Accessibility Standards
- WCAG 2.1 AA compliance
- Include alt text for all images
- Use proper heading structure
- Ensure sufficient color contrast
- Add ARIA labels where needed
- Test with keyboard navigation

## DecapCMS
- Config at `public/admin/config.yml`
- Collections point to `src/content/` folders
- Access admin at `/admin/`

## Quality Assurance Checklist
- [ ] Build completes: `npm run build`
- [ ] Tests pass: `npm test`
- [ ] All pages load without console errors
- [ ] Images load and are properly optimized
- [ ] Interactive components work (filters, maps)
- [ ] Mobile responsiveness verified
- [ ] SEO meta tags present and accurate

## Troubleshooting

### Common Astro Issues
- **Build fails**: Check for TypeScript errors or missing imports
- **Content not showing**: Verify collection schema in config.ts
- **Svelte component not interactive**: Add `client:load` directive

### SCSS Issues
- **Variables not found**: They're auto-imported via Vite config
- **@import warnings**: Use `@use` syntax instead

### Svelte 5 Issues
- **$: not working**: Use `$derived` rune instead
- **export let error**: Use `let { prop } = $props()` instead
