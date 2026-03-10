# ADR-0008: Frontend Stack - Static-First SvelteKit with SPA Auth Routes

## Status

Proposed

## Context

The frontend for the air quality sensor map app needs to stay operationally simple and low cost.

Current constraints and goals:

- Avoid running a separate SSR frontend server in production
- Keep deployment inexpensive (static hosting + CDN preferred)
- Backend is Go REST API; browser auth will be standardized on secure cookies
- SEO requirements are primarily for unauthenticated/public routes
- Authenticated routes are expected to be highly interactive (map filters, dashboards, user-specific views)

The team is considering Svelte and wants a deployment model that balances SEO, performance, and low infrastructure overhead.

## Decision

Adopt **SvelteKit** with a **static-first architecture**:

- Prerender public routes to static HTML for SEO
- Run authenticated sections as client-rendered SPA routes
- Do not deploy a Node/SSR runtime for frontend rendering in production
- Keep hosting provider-agnostic; frontend assets must be deployable to static hosting + CDN
- Keep the Go REST API as the only application server

Deployment phasing:

- Initial phase: serve built frontend static assets from the Go backend
- Future phase: move frontend static assets to dedicated static hosting/CDN when operationally useful, without changing app architecture

Routing model:

- Public prerendered routes: `/`, `/about`, `/how-it-works`, `/map`, and other SEO-relevant public pages
- Authenticated SPA routes: `/app/*`, `/account/*`, `/admin/*`, `/alerts/*`

Authentication model:

- Use HTTP-only secure cookies issued by backend for authentication/session handling
- Do not use browser-managed bearer token storage as the primary auth model

## Consequences

### Positive

- Lowest operational complexity among modern full-featured frontend options
- No separate SSR/frontend runtime to manage
- Good SEO for public content via static prerendered HTML
- Strong fit for interactive authenticated map workflows where CSR is sufficient
- Lower hosting cost and simpler deployment pipeline

### Negative

- No per-request SSR for public pages needing highly fresh server-rendered content
- Auth state is initialized client-side, which can introduce loading/flicker states if not handled carefully
- Deep-link routing for SPA paths depends on correct static host fallback configuration
- Cross-origin auth/cookie setup can become complex if frontend and API are on different domains/subdomains
- Large map and geospatial bundles can hurt initial load if not aggressively lazy-loaded

## Mitigations

- Configure static host rewrites/fallback so direct navigation to SPA routes resolves correctly
- Implement a deliberate authenticated app-shell loading state
- Lazy-load map libraries, geospatial tooling, heavy charting, and admin-only views
- Keep public SEO pages meaningful without JS by prerendering metadata and core content
- Standardize CORS/cookie policy early and prefer same-site deployment where practical

## Alternatives Considered

### Full SSR SvelteKit (server adapter)

Rejected for current needs. Adds runtime infrastructure and operational overhead with limited benefit for primarily public SEO pages plus highly interactive authenticated views.

### Pure SPA (no prerendered public routes)

Rejected for current needs. Simpler app model but weakens SEO for public pages and social sharing previews.

### Different frontend framework with similar static + SPA split

Not selected at this time. Could satisfy requirements, but SvelteKit is preferred for team fit and direct support for static prerender + SPA route behavior.

## Assumptions to Confirm

- Public route content is stable enough to prerender on build/deploy cadence
- SEO priority remains limited to unauthenticated routes
- Team accepts "Go API as only backend server" as a standing architectural constraint
