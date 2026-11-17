# PreVisit — Status

**Last updated:** 2026-11-17

## Done

- PRD v0.1 (`docs/product/PRD.md`)
- Business rules v0.2, all default values confirmed (`docs/business-rules.md`)
- `rails new previsit --database=postgresql --css=tailwind --skip-test` (Ruby 3.4.11, Rails 8.1.4)
- Repository pushed to GitHub
- `CLAUDE.md` and this status file

## In progress

- Development setup:
  - [x] Stop the PostgreSQL service inside WSL (avoid port 5432 conflict)
  - [x] Docker Compose for PostgreSQL and Redis
  - [x] `config/database.yml` points to the Docker database
  - [x] RSpec + FactoryBot
  - [x] Sidekiq as the Active Job adapter, Solid Queue removed
  - [x] GitHub Actions runs RSpec with PostgreSQL and Redis

## Next 3 tasks

1. Write `docs/architecture/data-model.md` (tables: users, doctors, availability_slots, appointments, intake_forms, visit_summaries)
2. Write 2–3 ADRs in `docs/adr/`: Sidekiq instead of Solid Queue, a provider-agnostic LLM layer with a free tier, Hotwire + ERB + Tailwind
3. Rails 8 authentication generator and the `patient` / `doctor` roles

## Milestones

| Version | Content | Target date |
|---|---|---|
| Setup | Docs, repo, CI, Docker Compose | 2026-11-29 |
| v0.1 | Accounts, roles, slots, booking with no double booking | 2026-12-06 |
| v0.2 | Intake form, AI pre-visit summary with Sidekiq | 2026-12-13 |
| v1.0 | Deployed, README, demo accounts | 2026-12-20 |

## Notes

- The local PostgreSQL inside WSL is disabled on purpose. The project uses PostgreSQL from Docker Compose, so anyone can run it with one command.
- The code lives in the WSL file system (`~/projects/previsit`), not on a Windows drive, for speed.
