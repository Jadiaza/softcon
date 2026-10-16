<!-- LOVABLE:BEGIN -->
> [!IMPORTANT]
> This project is connected to [Lovable](https://lovable.dev). Avoid rewriting
> published git history — force pushing, or rebasing/amending/squashing commits
> that are already pushed — as it rewrites history on Lovable's side and the
> user will likely lose their project history.
>
> Commits you push to the connected branch sync back to Lovable and show up in
> the editor, so keep the branch in a working state.
<!-- LOVABLE:END -->

# SoftContable — technical rules

- Module/submodule menu lives in `src/lib/navigation.ts`; generic `/$modulo` and `/$modulo/$sub` routes render any module not yet built, so new modules only need a config entry until they get a dedicated route.
- Dedicated screens (e.g. `contabilidad.puc.tsx`) are static routes that override the generic ones.
- Shared UI (PageHeader, KpiCard, StatusBadge, DataTable, ConfirmDialog, FormField) lives in `src/components/common`; reuse instead of duplicating.
- Demo data lives only in `src/services/adapters/mock/fixtures.ts` until a backend is connected; the legacy DynamicERP API endpoints are recorded per submodule in `navigation.ts` (`legacy`) for migration.
- Money/date formatting and NIT DV calculation go through `src/lib/format.ts` (es-CO).
- Architecture docs live in `docs/*_V2.md`; business rules belong in `src/domain` (pure, framework-free) so any backend can replicate them.
- Screens should consume data through `src/services` contracts (mock adapter today, HTTP later) instead of importing `mock.ts` directly.