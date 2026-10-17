Design a professional, modern UI for a multi-tenant B2B recruitment agency CRM. Dark-mode landing page, then a full app for logged-in agency users.

Tenancy & roles: Each tenant is a recruitment agency. Three roles with different visible actions: Admin (full access, team/settings management, final offer approval), Recruiter (manages candidates/clients/jobs, creates draft offers, cannot approve them), Coordinator (scheduling, limited edit access). Make role differences visible in the UI (e.g. a disabled/hidden "Approve" action for non-Admins).

Core entities to reflect accurately: Candidates (resume, application status, tags), Clients (employer companies, open job counts, activity/audit trail), Jobs, Interviews (schedule, panel feedback, timezone), Offers (draft vs approved status, client criteria thresholds), Audit logs (who did what, when).

Pages needed:

- **Landing page** — (dark mode), animated pipeline
<!-- - **Auth / organization setup** (sign in, register agency, pick tier) -->
- **Dashboard** — tenant selector, quick stats, "needs attention today" summary
- **Talent Pool** — candidate list with resume/status/tags
- **Client Ledger** — employer companies with open jobs + audit trail
- **Kanban Pipeline** — drag-and-drop stages: Source $\rightarrow$ Screening $\rightarrow$
- **Interview Scheduler** — upcoming interviews, feedback logs, timezone display
- **Offer Desk** — draft vs approved offer states, client threshold indicators, role-gated "Approve" action, visible audit trail (created by / approved by / timestamps)
- **Team & Settings** — invite recruiters, manage the three roles above
- **Billing page** — Free plan (no AI agent) vs Pro plan (AI agent included) clearly differentiated
- **AI Copilot sidebar ("Astra")** — slide-in drawer, chat-style, only visible/enabled for Pro-plan tenants; show an upgrade prompt for Free-plan users instead of the chat
