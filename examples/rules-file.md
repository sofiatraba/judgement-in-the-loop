# Example rules file

The `CLAUDE.md` I used on a five-day take-home test (a supplier lifecycle API from an OpenAPI contract, a dashboard, one-command boot). Company name and repository paths are generalised; everything else is as the agent read it at the start of every session.

```markdown
# Rules for this repository

Take-home test for a Product Engineer role.

## Scope and contract

1. The OpenAPI files in the brief are the single source of truth for the API. Never add, rename or extend endpoints or fields. Never edit the brief's files or the provided mock.
2. Deliver the brief's scope first. Improvements never contradict or replace required behaviour.
3. `docker compose up` from a clean clone must boot everything with Docker only. Every image builds from source. Verify before every merge to `main`.
4. Ports: backend `8080`, frontend `3000`, country mock `8088`. The database is not published on the host. Config via env vars with compose defaults.

## Domain and code

5. The domain layer has no framework imports. Rules about one aggregate live in that aggregate and are unit-tested there. Rules across aggregates (one pending candidacy, one supplier, never both) live in use cases and are backed by database constraints and locks.
6. Lock order in every transaction: DUNS lock when a candidacy is involved, then country lock before any supplier write, then rows. Never take them in another order.
7. Every business rule in the brief maps to at least one test named after the rule.
8. Internal supplier state keeps `ON_PROBATION`. Only the API mapper collapses it to `Active`.
9. Errors are `{ "info": "..." }`. `404` has an empty body. Status codes exactly as in `docs/decisions.md`. Spring's default error JSON must never reach a client.
10. Never validate countries against a real ISO list. The country service defines validity.
11. Score arithmetic uses integer/decimal maths, never `double`. Sorting uses the stored, rounded score.
12. Comments only where intent is not obvious, and short. No commented-out code, no TODOs on `main`.

## Docs

13. Docs are short, in plain language, with diagrams over prose. `SOLUTION.md` always reflects the current state.

## Git workflow

14. One feature per branch (`feat/…`, `chore/…`, `docs/…`). Conventional commits. Merge to `main` with `--no-ff`. Tests green before merging. Never commit directly to `main`.
```

What to notice:

- Rules 1 and 2 are about scope. They stopped the agent from "improving" the contract.
- Rule 6 did not exist on day one. An adversarial review found a deadlock; the fix became a rule so it could not come back.
- Rules 7 and 9 are testable. A rule the build cannot check is a wish.
- Rule 14 is the workflow. It gave the reviewers a history of small merges instead of one large commit.
