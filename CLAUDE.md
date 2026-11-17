# PreVisit — notes for Claude

## What this project is

PreVisit is a small clinic web app and a **portfolio project**. A patient books a 30-minute slot with a doctor and fills in a symptom form. A Sidekiq job sends the form to a free LLM and saves a short **pre-visit summary** for the doctor. The AI never diagnoses. The app uses **fake data only**.

The owner is a Rails developer who is **learning** with this project and will explain it in job interviews. Help them understand, not only finish.

## Read these first

| File | What it holds |
|---|---|
| `docs/STATUS.md` | What is done, what is in progress, the next tasks. **Start here.** |
| `docs/product/PRD.md` | Scope of v1.0 and the "Later" list |
| `docs/business-rules.md` | Rules `BR-xxx`. Code and tests must follow them. |
| `docs/architecture/` | Data model and architecture (when they exist) |
| `docs/adr/` | Decisions already made and why. Do not reverse them without asking. |

## Stack

- Ruby 3.4, Rails 8.1, PostgreSQL 17
- Hotwire (Turbo, Stimulus), ERB views, Tailwind CSS
- Sidekiq + Redis for background jobs (Active Job API: `perform_later`)
- Pundit for authorization, Rails 8 authentication generator
- `ruby_llm` with the Gemini free tier, wrapped in our own service class
- RSpec + FactoryBot, RuboCop, GitHub Actions
- PostgreSQL and Redis run with Docker Compose. Rails runs in WSL.

## Commands

```bash
docker compose up -d        # start PostgreSQL and Redis
bin/dev                     # start Rails, Tailwind watcher and Sidekiq
bundle exec rspec           # run all tests
bin/rubocop                 # lint
bin/rails db:prepare        # create and migrate the database
```

## How to work with the owner (learning rules)

| Kind of work | Who writes it | Claude's role |
|---|---|---|
| UI: views, layouts, Tailwind classes | Claude | Write it, then explain the Rails parts (forms, partials, Turbo, Stimulus, N+1 risks) |
| Business logic, models, validations | Owner | Give hints, ask questions, review |
| Sidekiq jobs, Redis | Owner | Give hints, ask questions, review |
| Tests for business rules | Owner | Suggest cases, review |
| Config, CI, Docker, boilerplate | Claude | Write it, explain briefly |

- Before writing code, explain the approach in a few lines and ask the owner to choose.
- When the owner writes code, point out mistakes and ask "why" questions.
- Use **simple English**. The owner is improving their English.
- Never write a long block of business logic without being asked.

## Scope rules

- Build only what is in the PRD scope for v1.0.
- A new idea, even a good one, goes to the "Later" section of `docs/product/PRD.md`. Do not build it.
- If a task is taking too long, suggest cutting scope, not adding time.

## Code conventions

- Each business rule has at least one test. Name the rule in the test, for example `describe "BR-020: one active appointment per slot"`.
- Prefer request specs over controller specs. Write policy specs for every Pundit policy.
- Enforce important rules in the database too (unique indexes, foreign keys, NOT NULL).
- Jobs must be idempotent (BR-057). Do not send names, emails or phone numbers to the LLM (BR-055).
- Store times in UTC. Show them in `Asia/Ho_Chi_Minh` (BR-014).
- Commit messages follow Conventional Commits: `feat:`, `fix:`, `test:`, `docs:`, `chore:`, `refactor:`.
- Never commit secrets. `config/master.key` stays out of Git.

## At the end of every session

Update `docs/STATUS.md`: what was finished, what is in progress, and the next 3 tasks. Then remind the owner to commit.
