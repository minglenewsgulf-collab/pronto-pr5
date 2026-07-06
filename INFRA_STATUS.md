# PRONTO PR - Infrastructure Status

Last updated: 2026-07-06

This document summarizes the current infrastructure state of the PRONTO PR repository. It is intended for future AI coding agents and for the product owner before starting new implementation tasks.

## 1. Infrastructure Already Done

### Frontend Foundation

- React 19 + TypeScript application exists.
- Routing uses TanStack Start / TanStack Router with file-based routes under `src/routes`.
- Build tooling uses Vite through `@lovable.dev/vite-tanstack-config`.
- UI component system is already present: Radix UI / shadcn-style components, Tailwind CSS 4, lucide-react icons, Recharts, Sonner.
- Existing UI, routes, and product screens are preserved.

### Supabase Client Foundation

- Supabase JS client dependency is present: `@supabase/supabase-js`.
- Supabase client helpers exist under `src/lib/supabase`.
- Server-side Supabase helpers exist under `src/server/supabase` for service-role-only backend work.
- Environment variable template exists in `.env.example`.
- Required frontend env variables:
  - `VITE_SUPABASE_URL`
  - `VITE_SUPABASE_ANON_KEY`
- Required server-only env variables for backend work:
  - `SUPABASE_URL`
  - `SUPABASE_SERVICE_ROLE_KEY`
- The service role key is not used in frontend code and must remain backend-only.

### Supabase Auth Layer

- Supabase Auth email/password service layer exists under `src/lib/auth`.
- Implemented auth functions:
  - `signUpWithEmail`
  - `signInWithEmail`
  - `signOut`
  - `getCurrentUser`
  - `getSession`
  - `onAuthSessionChange`
- Login/register/logout flows are connected to Supabase Auth.
- Session restoration after refresh is implemented through the store/auth listener.

### Data Layer Foundation

- Domain types and repository interfaces exist under `src/lib/data`.
- Supabase repository implementations exist for:
  - `campaigns`
  - `articles`
  - `reports`
  - `report_articles`
  - `exports`
  - `domain_metrics_cache`
- Current intended architecture is:

```text
UI / routes
  -> src/lib/store.tsx
  -> src/lib/data/supabaseAppData.ts
  -> src/lib/data/supabaseRepositories.ts
  -> Supabase
```

- Direct Supabase table calls have been moved out of UI/routes.
- Active `src` code no longer uses browser `localStorage` for the main campaign/report/export persistence path.

### Supabase Storage Foundation

- Storage helper layer exists under `src/lib/storage`.
- The app assumes a Supabase Storage bucket named `exports`.
- Helper functions support uploading, downloading, listing, and removing export files.
- The existing report export action is connected to Supabase Storage.
- Current export implementation creates an MVP placeholder `.xlsx` file, uploads it to the `exports` bucket, creates an `exports` table record, downloads the uploaded file, and saves it locally in the browser.

### SQL / Database Drafts

- Supabase migration drafts exist under `supabase/migrations`.
- Additional SQL helper/diagnostic scripts were created during setup under the workspace `outputs` folder.
- SQL work has covered:
  - initial PRONTO PR schema draft;
  - missing `reports`, `report_articles`, and `exports` table scripts;
  - `export_format` and `export_status` enum scripts;
  - relationship/RLS diagnostic script;
  - safe relationship/RLS fix script.

### External Metrics Foundation

- Backend-only domain metrics architecture exists under `src/server/domain-metrics`.
- Provider interfaces and stubs exist for:
  - Similarweb;
  - Semrush;
  - PR-CY.
- Provider classes now include real HTTP client skeletons with request URL builders, timeout handling, error handling, and response normalization.
- Providers still fall back to stable normalized mock metrics when API keys are missing or provider requests fail.
- Real API calls are server-side only and are not connected to UI yet.
- Resolver logic exists to:
  - check `domain_metrics_cache`;
  - return fresh cached metrics;
  - call a provider when cache is missing or stale;
  - save normalized metrics back to `domain_metrics_cache`.
- Article persistence now calls the domain metrics resolver through a TanStack Start server function.
- When available, `monthly_visits` feeds PREI `reach_score`, and `authority_score` feeds the PREI authority input.
- Article `prei_components.domain_metrics` records the metrics source, fetched timestamp, and normalized values used by PREI.
- If resolver/server env/provider access fails, article PREI falls back to current mock inputs.
- Server-only external API env variables are documented:
  - `SIMILARWEB_API_KEY`
  - `SEMRUSH_API_KEY`
  - `PRCY_API_KEY`
- These keys must never be prefixed with `VITE_` or imported into frontend/browser code.
- Provider endpoints, paid API plans, quotas, and exact response field mapping still need confirmation before production use.

### Vercel Deployment Configuration

- `vercel.json` exists.
- Current Vercel config:

```json
{
  "$schema": "https://openapi.vercel.sh/vercel.json",
  "installCommand": "bun install",
  "buildCommand": "NITRO_PRESET=vercel bun run build"
}
```

- `vite.config.ts` has been updated to target Vercel:

```ts
export default defineConfig({
  tanstackStart: {
    server: { entry: "src/server" },
  },
  nitro: {
    preset: "vercel",
  },
});
```

## 2. Partially Done

### Database Infrastructure

Partially done:

- Schema drafts exist.
- Some tables were already created manually in Supabase.
- The user reported that the first migration partially applied.
- At one point, confirmed existing tables were:
  - `campaigns`
  - `articles`
  - `domain_metrics_cache`
- Missing tables were later handled with simplified scripts:
  - `reports`
  - `report_articles`
  - `exports`
- Export enums were handled separately:
  - `export_format`
  - `export_status`

Still needs confirmation in Supabase:

- Exact live columns for every table.
- Whether all foreign keys exist.
- Whether all indexes exist.
- Whether RLS is enabled on every app table.
- Whether final policies match the intended ownership model.
- Whether `domain_metrics_cache` is fully backend-only.

### Reports / Campaign Persistence

Partially done:

- The app now has Supabase-backed repository paths for campaigns, reports, and exports.
- Creating a report creates Supabase records through the data layer.
- Exports are represented in the `exports` table.

Still transitional:

- Some adapter code supports multiple possible database shapes because the live Supabase schema was created through partial/manual scripts.
- Report snapshots still preserve prototype data shapes.
- Workspace publication analysis still uses deterministic prototype logic.

### Export Flow

Partially done:

- Export button is connected to Supabase Storage and the `exports` table.
- File upload and download helpers exist.
- Export format is `.xlsx`.

Still MVP placeholder:

- The generated `.xlsx` is a placeholder Blob with CSV-style content, not a true Excel workbook.
- UI text may still refer to PDF because the design was intentionally not changed.
- Export is not yet based on final PREI/report aggregation logic.

### Auth

Partially done:

- Supabase email/password auth is wired.
- Login/register/logout service functions exist.
- Session restoration is implemented.

Still missing:

- Password reset.
- Email confirmation UX.
- Profile/user settings table if needed.
- Production-grade auth error copy.
- Final decision on signup UX, because the existing Lovable signup modal did not originally include a separate password field.

## 3. Still Missing

### Backend / Supabase

- Final confirmed Supabase schema documentation from the live database.
- Fully validated foreign keys in production.
- Fully validated RLS policies in production.
- Supabase Edge Functions.
- Production backend endpoint or job that calls the domain metrics resolver.
- Confirmed production external metrics provider contracts.
- Article URL ingestion backend.
- Link checking job for `broken` / `missing` article statuses.
- True report snapshot locking.
- True XLSX generation.

### Product Data Pipeline

- Real URL extraction.
- Real article metadata parser.
- Screenshot generation/storage.
- Real external traffic/authority API calls with confirmed provider contracts.
- Production domain metrics refresh scheduling.
- Similarweb integration beyond the current HTTP skeleton.
- Semrush / PR-CY integration beyond the current HTTP skeletons.
- AI/LLM structured JSON pipeline.
- AI output validation.
- PREI v1.0 calculation implementation.
- Final article status transitions.

### Frontend/Product

- Remove remaining prototype/mock publication analysis.
- Render publication cards from real saved article rows.
- Add real loading/processing/ready states.
- Report history based on final report snapshot rules.
- Dashboard metrics based on real database aggregates.
- Error/empty states for production data.

### Engineering

- Automated test setup.
- CI build/lint checks.
- Verified local development instructions for a non-Lovable environment.
- Final production deployment guide.
- Cleanup of duplicated root-level Lovable export files if they are confirmed unused.

## 4. Current Known Deployment Issue: Vercel / Lovable

Current status:

- Vercel deployment from GitHub has reached `Ready`.
- The root route `/` previously returned Vercel `404 NOT_FOUND`.
- After deployment config changes, the login page opened.
- The latest reported issue is that login/register buttons on the deployed Vercel page do not respond.

Likely issue:

- The current symptom looks like a client-side JavaScript or hydration problem, not only a login form bug.
- The server-rendered HTML appears to load, but client event handlers may not be running.

Known related facts:

- Lovable/TanStack config originally appeared to default Nitro to a Cloudflare-style target.
- Vercel needs the Nitro preset to be `vercel`.
- `vite.config.ts` currently sets:
  - `tanstackStart.server.entry = "src/server"`
  - `nitro.preset = "vercel"`
- `vercel.json` currently forces:
  - `NITRO_PRESET=vercel bun run build`
- The repository contains duplicate root-level Lovable export files and canonical `src/` files. Active TypeScript config appears focused on `src`, but the duplicate root files remain an infrastructure risk until deployment behavior is fully confirmed.

What to check next:

- Browser console errors on the deployed Vercel URL.
- Whether client JS bundles return `200`.
- Whether hydration starts successfully.
- Whether Vercel is serving the correct Nitro client/server output.
- Whether the active route tree is the `src/routeTree.gen.ts` version.
- Whether root-level duplicate files are interfering with TanStack Start build output.

Do not change UI while investigating this issue.

## 5. Current Supabase Status

### Confirmed In Code

- Supabase client exists.
- Supabase Auth is used by login/register/logout.
- Data repositories target Supabase tables.
- Storage helpers target the `exports` bucket.
- Export flow writes to Storage and the `exports` table.

### Confirmed From User Reports

- Supabase database was partially migrated manually.
- `campaigns`, `articles`, and `domain_metrics_cache` existed after the first interrupted migration.
- Missing report/export tables and export enums were addressed through follow-up SQL scripts.
- The database was considered "ready enough" by the user for current export work.

### Still Needs Manual Supabase Dashboard Verification

- `VITE_SUPABASE_URL` is set in Vercel.
- `VITE_SUPABASE_ANON_KEY` is set in Vercel.
- Email/password provider is enabled in Supabase Auth.
- Storage bucket `exports` exists.
- Storage bucket policies allow authenticated users to access only their own export files, or access is otherwise intentionally managed.
- RLS is enabled on:
  - `campaigns`
  - `articles`
  - `reports`
  - `report_articles`
  - `exports`
  - `domain_metrics_cache`
- `domain_metrics_cache` is not readable/writable by normal authenticated frontend users.
- Foreign keys exist between:
  - `articles -> campaigns`
  - `reports -> campaigns`
  - `report_articles -> reports`
  - `report_articles -> articles`
  - `exports -> reports`

## 6. Current Data Flow

### Authentication

```text
Login / Signup UI
  -> src/lib/store.tsx
  -> src/lib/auth/service.ts
  -> Supabase Auth
```

### Campaigns / Reports / Exports

```text
UI / routes
  -> src/lib/store.tsx
  -> src/lib/data/supabaseAppData.ts
  -> src/lib/data/supabaseRepositories.ts
  -> Supabase tables
```

### Export Storage

```text
Report page export button
  -> src/lib/storage/reportExport.ts
  -> src/lib/storage/exports.ts
  -> Supabase Storage bucket: exports
  -> Supabase table: exports
```

### Publication Analysis

Current state:

```text
Workspace URL input
  -> deterministic prototype analysis
  -> in-memory/store publication shape
  -> report snapshot / export bridge
```

Not final:

- URL/domain parser exists, but real article extraction is not implemented yet.
- Article metadata extraction service exists, but it still derives metadata from the current mock publication object.
- External metrics provider architecture exists. Providers have real HTTP skeletons, return mock metrics while API keys are missing, and feed article PREI through the server-side resolver.
- No AI pipeline yet.
- No PREI v1.0 formula yet.

## 7. Known Technical Risks

1. Vercel hydration/runtime issue is not fully resolved.
   - Priority: P0
   - Reason: deployed app can render but buttons may not work.

2. Live Supabase schema may not exactly match TypeScript repository types.
   - Priority: P0
   - Reason: migration was partially applied and then resumed manually.

3. RLS/storage policies need live verification.
   - Priority: P0
   - Reason: incorrect policies can either block the app or expose user data.

4. Root-level duplicate files may confuse future agents or build tooling.
   - Priority: P1
   - Reason: active code appears to live in `src`, but root duplicates are stale.

5. Export file is not a real workbook yet.
   - Priority: P1
   - Reason: acceptable for MVP plumbing, not final user-facing export.

6. Prototype publication analysis still exists.
   - Priority: P1
   - Reason: users can create reports, but report content is not yet real backend-derived analysis.

7. No automated tests or CI checks are confirmed.
   - Priority: P2
   - Reason: infrastructure changes are currently verified manually.

## 8. Next Recommended Product Tasks

Recommended order:

1. Fix Vercel client-side runtime/hydration.
   - Why: deployed users must be able to click login/signup and navigate.
   - Depends on: deployed URL, browser console/network inspection.

2. Verify live Supabase schema against repository assumptions.
   - Why: current app code contains compatibility fallbacks because the schema was partially/manual-created.
   - Depends on: read-only SQL diagnostic from Supabase SQL Editor.

3. Confirm and finalize RLS/storage policies.
   - Why: user data isolation is required before real beta usage.
   - Depends on: live schema confirmation.

4. Remove schema compatibility fallbacks after database shape is finalized.
   - Why: cleaner and safer data layer before feature work.
   - Depends on: finalized live schema.

5. Implement real article URL persistence.
   - Why: publication cards must come from the database, not prototype state.
   - Depends on: stable `campaigns` and `articles` tables.

6. Add URL/domain normalization and duplicate handling.
   - Why: same URL should not duplicate inside one campaign.
   - Depends on: article persistence.

7. Add real report creation rules.
   - Why: reports should snapshot active articles and preserve historical results.
   - Depends on: article persistence and report/article relationship.

8. Implement PREI v1.0 calculation.
   - Why: current score is prototype data, not the final approved formula.
   - Depends on: article metadata, sentiment, placement, media type, reach/authority inputs.

9. Replace external metrics stubs with real provider calls.
   - Why: PREI requires real reach/authority/domain metrics.
   - Depends on: server-side API keys, backend execution path, provider contracts.

10. Replace placeholder export with true XLSX generation.
    - Why: current export proves infrastructure, but final users need a real Excel file.
    - Depends on: final report snapshot and metrics shape.

## 9. Rules For Future Work

- Do not redesign UI unless explicitly requested.
- Do not change routes unless the task requires it.
- Do not use Supabase service role key in frontend code.
- Keep data access behind repositories.
- Keep product logic out of UI components where practical.
- Do not physically delete articles for normal users; use article status `removed`.
- Keep `domain_metrics_cache` backend-only.
- Prefer small safe tasks over large refactors.
- Before deleting root-level duplicate files, confirm they are not used by Lovable/TanStack/Vercel.
