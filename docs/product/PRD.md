# PreVisit — Product Requirements Document (PRD)

| | |
|---|---|
| **Version** | 0.1 (draft) |
| **Date** | 2026-11-15 |
| **Owner** | [Your Name] |
| **Status** | Draft — under review |

---

## 1. Summary

PreVisit is a web app for a small clinic. A patient books an appointment with a doctor and fills in a short symptom form before the visit. An AI model turns the form into a short **pre-visit summary** for the doctor. The doctor reads the summary before the visit and starts the visit already knowing the main problem.

PreVisit **does not diagnose** and **does not give medical advice**. It only organizes what the patient wrote.

> This is a portfolio project. It uses **fake data only**.

## 2. Problem

- Doctors often spend the first 5–10 minutes of a visit asking basic questions: what is the problem, when did it start, what medicine the patient takes.
- Patients forget important details when they are in front of the doctor.
- Paper forms in the waiting room are slow, and the doctor reads them only at the start of the visit.

## 3. Users

| User | Description | Main need |
|---|---|---|
| **Patient** | A person who wants to see a doctor at the clinic | Book a visit easily and tell the doctor about the problem before the visit |
| **Doctor** | A doctor who works at the clinic | Know the patient's main problem before the visit starts |

There is **no clinic admin** in v1.0. Doctor accounts are created by a seed script.

## 4. Goals and non-goals

**Goals (v1.0)**

1. A patient can book a free 30-minute slot without double booking.
2. A patient can submit a symptom form for each appointment.
3. A doctor can read an AI-generated summary of the form before the visit.
4. Each user can see only the data they are allowed to see.

**Non-goals (v1.0)**

- Diagnosis, treatment advice, or any medical decision by AI.
- Chat with AI.
- Payments, insurance, video calls.
- Real patient data.

## 5. Scope (v1.0 features)

| ID | Feature | Short description |
|---|---|---|
| F-01 | Accounts and roles | Patient sign up, log in, log out. Two roles: patient and doctor. |
| F-02 | Doctor availability | A doctor creates 30-minute time slots and deletes slots that are not booked. |
| F-03 | Booking | A patient sees free slots, books one, and can cancel under the cancellation rule. |
| F-04 | Intake form | After booking, a patient fills in a symptom form for that appointment. |
| F-05 | AI pre-visit summary | A background job sends the form to an LLM and saves a short summary. |
| F-06 | Doctor schedule | A doctor sees upcoming appointments and opens each summary. |

The exact rules for each feature are in [`business-rules.md`](../business-rules.md).

## 6. User stories and acceptance criteria

### F-01 Accounts and roles

**US-01** As a patient, I want to create an account, so that I can book appointments.

- Given I am not logged in, when I sign up with a valid email and password, then I am logged in as a patient.
- A user cannot choose the doctor role when signing up. (BR-002)

**US-02** As a user, I want to log in and log out, so that my data is safe.

### F-02 Doctor availability

**US-03** As a doctor, I want to create time slots, so that patients can book them.

- Each slot is 30 minutes and starts at :00 or :30. (BR-010)
- I cannot create a slot in the past or more than 30 days ahead. (BR-011)
- I cannot create slots that overlap. (BR-012)

**US-04** As a doctor, I want to delete a slot that nobody booked, so that I can change my schedule.

- I cannot delete a booked slot. (BR-013)

### F-03 Booking

**US-05** As a patient, I want to see free slots for a doctor, so that I can choose a time.

**US-06** As a patient, I want to book a slot, so that I have an appointment.

- If two patients book the same slot at the same time, only one succeeds. The other sees a clear message. (BR-020)
- I cannot book a slot that starts in less than 1 hour. (BR-021)

**US-07** As a patient, I want to cancel my appointment, so that the slot is free for others.

- I can cancel only if the visit starts in 2 hours or more. (BR-030)
- After I cancel, the slot is free again. (BR-031)

**US-08** As a patient, I want to see my appointments, so that I remember my visits.

### F-04 Intake form

**US-09** As a patient, I want to describe my symptoms before the visit, so that the doctor understands my problem.

- I should submit the form at least 1 hour before the visit. If I submit later, the form is still accepted but marked as late. (BR-042)
- I cannot submit or edit the form after the visit starts. (BR-043)
- If my form mentions emergency signs, I see a message to call 115 right away. (BR-054)

### F-05 AI pre-visit summary

**US-10** As a doctor, I want a short summary of the patient's form, so that I can prepare for the visit quickly.

- The summary only restates what the patient wrote. It has no diagnosis and no treatment advice. (BR-053)
- The summary is labeled "AI-generated — please confirm with the patient." (BR-056)
- If the summary fails, I can see the failed status and the original form. (BR-051, BR-052)

### F-06 Doctor schedule

**US-11** As a doctor, I want to see my upcoming appointments with the summary status, so that I know which patients are ready.

## 7. AI behavior and safety

- The AI **summarizes only**. It must not diagnose, guess a disease, or suggest medicine.
- The emergency check is a **simple keyword rule in code**, not an AI decision. It works even when the AI service is down. (BR-054)
- We send **only the symptom form** to the LLM. No name, email, phone, or date of birth. (BR-055)
- The demo uses a **free LLM tier**. Free tiers may use the data to improve their products, so the app uses fake data only. (BR-060)
- A real product would need a paid plan with a data processing agreement and a medical review of the prompt.

## 8. Success metrics (for the demo)

| Metric | Target |
|---|---|
| Double bookings in tests (including two requests at the same time) | 0 |
| Summaries that are ready within 2 minutes after the form is submitted | ≥ 90% |
| Summaries with a diagnosis or treatment advice (checked on 10 sample forms) | 0 |
| Users who can see another user's data (checked by policy tests) | 0 |

## 9. Assumptions and constraints

- One clinic, one time zone: **Asia/Ho_Chi_Minh**. Times are stored in UTC.
- The user interface is in English.
- The free LLM tier has daily limits, so the app must handle rate-limit errors (HTTP 429).
- Built by one developer in about 4 weeks next to a job search.

## 10. Risks and mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| AI writes a diagnosis or advice | Patient safety, trust | Strict prompt, doctor-only summary, label "AI-generated", test with 10 sample forms |
| Free LLM tier limit is reached | Summary not created | Retry with backoff, `failed` status, doctor can retry, original form is always visible |
| Two patients book the same slot | Broken schedule | Unique constraint in the database, not only a check in Ruby |
| Users see other users' data | Privacy problem | Pundit policies and policy tests for every action |
| Project scope grows | Not finished on time | Anything new goes to "Later" (section 11) |

## 11. Later (out of scope for v1.0)

- Chat with AI instead of a form
- Email or SMS reminders
- Clinic admin role, doctor accounts managed in the UI
- Doctor cancels or moves an appointment
- Encryption of single database fields, full audit log
- Several LLM providers with automatic fallback, local model (Ollama)
- Vietnamese and Japanese user interface
- Mobile app and public API

## 12. Open questions

- [ ] Default values marked "(default)" in `business-rules.md` need to be confirmed.

## 13. Glossary

| Term | Meaning |
|---|---|
| **Slot** | A 30-minute time when a doctor is free to see a patient |
| **Appointment** | A slot booked by a patient |
| **Intake form** | The symptom form a patient fills in before the visit |
| **Pre-visit summary** | The short AI-generated text for the doctor, made from the intake form |
| **Red flag** | A symptom that may mean an emergency, for example chest pain |
