# Week 26 Weekend: Polish Projects 5 and 6, Harden Deploy

A polish weekend. No new features. Make both projects demo-ready and the deploy pipeline reliable.

**Estimated time:** 5-6 hours.

**Deadline:** Monday before the Capstone Sprint.

---

## Project 5 Polish (Booking System)

- [ ] A public booking page at `/book/[providerId]` that shows free slots for the next 7 days.
- [ ] A customer can book without creating an account (phone is identity).
- [ ] Reminder SMS goes out 24 hours before the appointment.
- [ ] Cancellation link in the reminder message.
- [ ] Admin page for providers to manage their availability windows.
- [ ] A single test case covers: find slots -> book -> try to book same slot again (fails) -> cancel -> slot is free again.

## Project 6 Polish (Notification Hub)

- [ ] `notify()` works for every app already built (shop, chama, booking).
- [ ] Every existing direct-send call is replaced with `notify()`.
- [ ] Quiet hours tested against a real user.
- [ ] Fallback from WhatsApp to SMS works (deliberately break WhatsApp to verify).
- [ ] Admin page shows last 24h delivery stats.
- [ ] `notify({ urgent: true })` bypasses quiet hours.

## Deploy Pipeline Polish

- [ ] CI runs on every push; green check required for merge.
- [ ] Docker images publish on push to main.
- [ ] Deploy workflow pulls latest images and restarts containers.
- [ ] Rollback procedure documented in RUNBOOK.md.
- [ ] Health checks on every service.
- [ ] Alerts fire when a service is unhealthy for more than 5 minutes.

---

## Deliverables

- Both projects running in production (or demoable on localhost).
- A green CI/CD pipeline.
- A RUNBOOK.md with: how to deploy, how to roll back, how to view logs, how to restart a stuck service.
- Screenshots of both admin dashboards.

---

## Grading Rubric (100 pts -- final pre-capstone grade)

| Area | Points |
|---|---|
| Project 5 end-to-end working | 25 |
| Project 6 end-to-end working | 25 |
| CI pipeline green | 10 |
| Deploy pipeline automated | 10 |
| RUNBOOK.md quality | 10 |
| Integration with earlier projects (shop, chama) uses notification hub | 10 |
| Alerts work | 10 |

---

## Hints

**Resist new features.** This is about polish and integration, not greenfield.

**Dogfooding.** Make sure Project 6 is actually used by Project 3 (the shop) and Project 4 (the chama). If it is not, replace the direct sends.

**Dev fire drills.** Do a full deploy and rollback before the weekend ends. Time yourself. Under 10 minutes is good; under 5 is excellent.

**Runbook is for future you.** Write it like you have forgotten everything. When the production site breaks at 11pm, a well-written runbook saves you.

---

## Preparing For The Capstone

Monday starts the Capstone Sprint: four weeks to build The African SME OS. Before Monday:

1. Read `week27/day1` to see what Week 27 expects.
2. Think about whether you want to bring your own aligned startup idea instead of the SME OS.
3. Back up your work. Tag the repo `pre-capstone` so you can always return to it.

The capstone is different from the weekly structure: more self-directed, more design, more you driving. Come in fresh.
