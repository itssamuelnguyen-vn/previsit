# PreVisit — Business Rules

| | |
|---|---|
| **Version** | 0.2 |
| **Date** | 2026-11-17 |
| **Related** | [PRD](product/PRD.md) |

Each rule has an ID (`BR-xxx`). Code, tests, pull requests and issues refer to these IDs.
Every rule must have at least one automated test.

---

## 1. Accounts and access

| ID | Rule |
|---|---|
| BR-001 | There are two roles: `patient` and `doctor`. Each user has exactly one role. |
| BR-002 | Patients sign up by themselves. Doctor accounts are created only by the seed script in v1.0. Nobody can choose the doctor role in the sign-up form. |
| BR-003 | A patient can see and change only their **own** appointments and intake forms. |
| BR-004 | A doctor can see only appointments booked **with them**, and the intake forms and summaries of those appointments. |
| BR-005 | Patients **cannot** see the AI summary. The summary is for doctors only. |

## 2. Time slots

| ID | Rule |
|---|---|
| BR-010 | A slot is always **30 minutes**. It starts at minute :00 or :30. |
| BR-011 | A doctor can create slots only in the future and at most **30 days** ahead. |
| BR-012 | Slots of the same doctor cannot overlap. |
| BR-013 | A doctor can delete a slot only if it has no active appointment. |
| BR-014 | All times are stored in UTC and shown in the clinic time zone `Asia/Ho_Chi_Minh`. |

## 3. Booking

| ID | Rule |
|---|---|
| BR-020 | A slot can have **at most one active appointment**. This is enforced by a unique constraint in the database, so it also works when two requests arrive at the same time. |
| BR-021 | A patient can book a slot only if it starts in **1 hour or more**. |
| BR-022 | A patient can have at most **3 upcoming active appointments** at the same time. |
| BR-023 | A patient cannot have two active appointments at the same time, even with different doctors. |
| BR-024 | Appointment status is `booked` or `cancelled`. Appointments are never deleted. They stay for history. |

## 4. Cancellation

| ID | Rule |
|---|---|
| BR-030 | A patient can cancel only if the visit starts in **2 hours or more**. If it is less than 2 hours, the app shows: "It is too late to cancel online. Please call the clinic." |
| BR-031 | When an appointment is cancelled, its slot becomes free and other patients can book it. |
| BR-032 | A cancelled appointment cannot be booked again. The patient makes a new booking. |

## 5. Intake form

| ID | Rule |
|---|---|
| BR-040 | Each appointment has at most **one** intake form. |
| BR-041 | Form fields: **main symptom** (required, max 500 characters), **when it started** (required), **severity from 1 to 10** (required), **current medicines** (optional), **allergies** (optional), **other notes** (optional, max 1,000 characters). |
| BR-042 | The patient should submit the form **at least 1 hour** before the visit. A form submitted later is still accepted, but it is marked `late` and the doctor sees this mark. |
| BR-043 | After the visit start time, the form cannot be submitted or edited. |
| BR-044 | Before the visit starts, the patient can edit the form. Each edit creates a new summary that replaces the old one. |
| BR-045 | If an appointment is cancelled, its form can no longer be edited. |

## 6. AI pre-visit summary

| ID | Rule |
|---|---|
| BR-050 | A summary is created by a **background job** after the form is submitted or edited. The patient does not wait for the AI. |
| BR-051 | Summary status is `pending`, `ready` or `failed`. The doctor always sees the status and the original form. |
| BR-052 | If the LLM call fails (timeout, HTTP 429 or 5xx), the job retries up to **3 times** with backoff. After that the status is `failed`, and the doctor can press "Retry". |
| BR-053 | The summary only restates what the patient wrote, in this order: main symptom, when it started, severity, medicines, allergies, notes. It **must not** include a diagnosis, a possible disease, or treatment advice. |
| BR-054 | **Red flags** are checked by a keyword rule in code, **not by AI**. Red flags: chest pain, trouble breathing, fainting, heavy bleeding. If the form mentions a red flag, the patient immediately sees "If this is an emergency, call 115 now", and the doctor sees a "Possible urgent symptoms" mark. |
| BR-055 | Only the form content is sent to the LLM. **No** name, email, phone number or date of birth. |
| BR-056 | Every summary is shown with the label: "AI-generated — please confirm with the patient." |
| BR-057 | The job is **idempotent**: running it twice for the same form version does not create two summaries. |

## 7. Data and privacy

| ID | Rule |
|---|---|
| BR-060 | The app uses **fake data only**. Every page shows a banner: "Demo app — fake data only — not medical advice." |
| BR-061 | Sensitive fields (form content, summary, password) are filtered from logs and error reports. |

---

## Change log

| Version | Date | Change |
|---|---|---|
| 0.2 | 2026-11-17 | Confirmed all default values (BR-011, BR-021, BR-022, BR-041, BR-052, BR-054) |
| 0.1 | 2026-11-15 | First draft |
