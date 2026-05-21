# Loan Pre-Screen Service — Coding Exercise

Welcome. This is a focused backend task. You have ~2 hours.

We are more interested in **how you think and structure things** than in finishing every line. Quality of decisions matters more than lines of code.

## What you'll build

A small HTTP service with one endpoint that takes a loan application and returns a pre-screen decision — one of `AUTO_APPROVE`, `REFER_TO_HUMAN`, `AUTO_DECLINE` — plus a short rationale.

### Input

A loan application has:

- `amount` (positive number) — requested amount in EUR
- `term_months` (positive integer) — term in months
- `monthly_income` (positive number) — applicant's monthly income in EUR
- `monthly_debt` (non-negative number) — applicant's existing monthly debt payments in EUR
- `occupation` (string) — applicant's stated occupation
- `reason_for_loan` (string, free text) — applicant's stated reason

### Deterministic rules (apply first)

Starting from a default decision of `AUTO_APPROVE`:

- If `amount > 100_000` → `REFER_TO_HUMAN`
- If `term_months > 84` → `AUTO_DECLINE`
- If `(monthly_debt + amount / term_months) / monthly_income > 0.5` → `AUTO_DECLINE`
- If `occupation` is in `{"unemployed", "self-declared crypto trader", "professional gambler"}` → `AUTO_DECLINE`

If multiple rules match, the strictest wins (`AUTO_DECLINE` > `REFER_TO_HUMAN` > `AUTO_APPROVE`).

### LLM-based check (must use a real local LLM)

Send `reason_for_loan` to a **local LLM** running on your machine via Ollama, LM Studio, or any equivalent that exposes an OpenAI-compatible HTTP API on `localhost`. No cloud calls — the whole exercise runs locally.

The LLM must return a **structured** result containing at least:

- `red_flag` (boolean)
- `category` (one of `"gambling"`, `"crypto"`, `"debt_consolidation"`, `"other"`, or `null` if no red flag)
- `rationale` (short string)

How you guarantee the LLM produces this shape (`response_format`, JSON schema, prompt engineering + post-validation, …) is up to you. Small local models can be unreliable at strict JSON output — handle that.

If `red_flag` is `true`, **downgrade** the decision one step:

- `AUTO_APPROVE` → `REFER_TO_HUMAN`
- `REFER_TO_HUMAN` → `AUTO_DECLINE`
- `AUTO_DECLINE` stays `AUTO_DECLINE`

The final response combines the decision and a short rationale (deterministic and/or LLM-sourced).

## Required constraints

- **Language: Python ≥3.11.** Web framework: your choice (FastAPI, Flask, Django, Litestar — whatever you prefer).
- **Real local LLM.** Use Ollama, LM Studio, or any OpenAI-compatible local server. Any small instruct model works (Llama 3.2, Qwen 2.5, Phi-3.5, …).
- **At least one unit test for the business decision logic that runs without the LLM service** — no real LLM call, no live process required. How you achieve this is up to you; we are very interested in your approach to testing LLM-dependent code.
- **A clear contract for consumers.** Someone calling your service should be able to figure out how to call it and what to expect back, without reading your source.

## Setup

```bash
git clone <REPO_URL>
cd <REPO>
# Use whatever package manager you like: uv, pip, poetry, ...
```

You should have a local LLM server running by the time we start (we asked you to set this up beforehand). Verify it responds to a request before you begin. If you hit any local-setup blocker, flag it at the start — we have an OpenAI cloud key with small credit ready as a fallback.

## How to share your work

Commit as you go and push to a branch — we'll watch the commit history. Incremental commits are encouraged; we are not grading commit-message poetry, but we like seeing how your thinking evolved.

## Stretch goals (only after the core task is solid)

If you finish with time to spare, pick **one** — don't try all three:

1. **Resilience.** Make the LLM call robust to slow responses, timeouts, and transient errors.
2. **Input validation contract.** Return clear, structured error responses for invalid inputs.
3. **LLM-seam test.** Add a test that exercises how your service handles different LLM responses (e.g., asserts behavior when `red_flag = true`).

## Out of scope

You do **not** need to: deploy, set up a database, add auth, write a Dockerfile, write a frontend, or handle concurrent users. Keep it local and focused.

## What we're paying attention to

- How you structure code as the task gets bigger.
- How you compose deterministic rules with an LLM call.
- How you make LLM-dependent code testable.
- How you communicate decisions and trade-offs as you go.
- How you use your AI assistant — please do use it. We are curious how you collaborate with it, not whether you use it.

## If you're stuck or unsure

Ask us. The spec has intentional ambiguity in some places — your questions tell us as much as your code does.

Good luck.
