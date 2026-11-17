# PreVisit — Status

**Last updated:** 2026-11-17

## Done

- PRD v0.1 (`docs/product/PRD.md`)
- Business rules v0.2, all default values confirmed (`docs/business-rules.md`)
- `rails new previsit --database=postgresql --css=tailwind --skip-test` (Ruby 3.4.11, Rails 8.1.4)
- Repository pushed to GitHub
- `CLAUDE.md` and this status file
- Development setup (PR #5, merged 2026-11-17):
  - PostgreSQL service inside WSL stopped (avoids a port 5432 conflict)
  - Docker Compose for PostgreSQL 17 and Redis 7 (`compose.yml`)
  - `config/database.yml` points to the Docker database
  - RSpec + FactoryBot
  - Sidekiq as the Active Job adapter, Solid Queue removed
  - GitHub Actions runs RSpec with PostgreSQL and Redis

## In progress

- Nothing. The setup milestone is finished.

## Next 3 tasks

1. Write `docs/architecture/data-model.md` (tables: users, doctors, availability_slots, appointments, intake_forms, visit_summaries)
2. Write 2–3 ADRs in `docs/adr/`: Sidekiq instead of Solid Queue, a provider-agnostic LLM layer with a free tier, Hotwire + ERB + Tailwind
3. Rails 8 authentication generator and the `patient` / `doctor` roles

## Milestones

| Version | Content | Target date |
|---|---|---|
| Setup | Docs, repo, CI, Docker Compose | 2026-11-29 (done 2026-11-17) |
| v0.1 | Accounts, roles, slots, booking with no double booking | 2026-12-06 |
| v0.2 | Intake form, AI pre-visit summary with Sidekiq | 2026-12-13 |
| v1.0 | Deployed, README, demo accounts | 2026-12-20 |

## Notes

- The local PostgreSQL inside WSL is disabled on purpose. The project uses PostgreSQL from Docker Compose, so anyone can run it with one command.
- The code lives in the WSL file system (`~/projects/previsit`), not on a Windows drive, for speed.
- CI fix: the first `test` job run failed at `bin/rails db:test:prepare` because `db/schema.rb` did not exist yet (there are no migrations). The fix commits the empty initial `db/schema.rb`. Keep `db/schema.rb` in Git; CI loads it.
- Branch protection: the `protect-develop` ruleset protects the default branch (`develop`). It blocks deletion and force pushes, requires a pull request and a linear history, and requires the `test`, `lint`, `scan_js` and `scan_ruby` checks to pass. All work goes through a branch and a PR. To rewrite history on `develop`, the ruleset must be turned off for a moment and then turned on again.
