# BlindLaser

Blank static-site starter for a simple browser game.

- SvelteKit 3 + Svelte 5 (runes)
- Tailwind CSS v4 (via `@tailwindcss/vite`)
- TypeScript, Prettier
- `@sveltejs/adapter-static` — fully prerendered, no server
- GitHub Pages deployment workflow

## Scripts

```sh
npm install
npm run dev        # http://localhost:5173
npm run build      # static output in build/
npm run preview    # preview the build
npm run check      # type-check Svelte + TS
npm run format     # prettier
```

## GitHub Pages

1. Create a repository on GitHub and push this project to `main`.
2. In the repo: **Settings → Pages → Build and deployment → Source: GitHub Actions**.
3. The workflow at `.github/workflows/deploy.yml` builds and deploys on every push.

The workflow sets `BASE_PATH` automatically:

- project site (`<user>.github.io/<repo>`) → base `/<repo>`
- user site (`<user>.github.io`) → base ``

To reproduce a project-site build locally:

```sh
BASE_PATH=/BlindLaser npm run build
npm run preview
```

Keep root-relative links/assets base-aware by using `resolve()` / `asset()` from
`$app/paths` (or route links via `href`, which SvelteKit prefixes for you).

## Structure

```
src/
  app.html
  routes/
    +layout.svelte
    +layout.ts       # prerender = true, trailingSlash = 'always'
    +page.svelte     # placeholder home page
    layout.css       # @import 'tailwindcss';
static/              # copied as-is; add favicon, images, etc.
```

## Suggested next steps (not included)

- **UI**: `bits-ui` (or `shadcn-svelte`) for accessible dialogs/menus, `lucide-svelte`
  for icons, `@fontsource-variable/*` for self-hosted fonts.
- **Game rendering**: Canvas 2D for simple games; PixiJS if you need WebGL,
  sprites, and particles at scale.
- **Game loop**: keep engine code in `src/lib/game/` and drive it from a
  `requestAnimationFrame` loop in one Svelte component.
- **Audio**: `howler.js` or a small Web Audio helper for synthesized SFX.
- **Feel**: screen shake, hit-stop, particles, and sound matter more than art at first.
- **Saves/options**: `localStorage`, plus a PWA plugin if you want installability.
