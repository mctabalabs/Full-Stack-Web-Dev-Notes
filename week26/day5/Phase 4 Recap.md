# Week 26, Day 5: Phase 4 Recap

Phase 4 is done. You have crossed the line from "writes code" to "ships production-grade systems". This week closes 26 weeks of content; the Capstone Sprint starts Monday.

---

## What Phase 4 Covered

| Week | Topic | Deliverable |
|---|---|---|
| 23 | BullMQ queues | Notifications through a real queue |
| 24 | Microservices split | Notification service as its own process |
| 25 | Docker + Nginx + TLS | Full stack deployable via compose |
| 26 | CI/CD + Projects 5/6 | Booking system + notification hub + automated deploys |

Eleven new concepts. Zero regressions in earlier work.

---

## Phase 4 Roadmap Check

From the syllabus:

| Syllabus bullet | Where |
|---|---|
| Queues: Redis/BullMQ | Week 23 |
| Microservices: split Notification Service from Core App | Week 24 |
| DevOps: Docker + CI/CD pipelines (GitHub Actions) | Weeks 25-26 |
| Project 5: Booking & Appointments System | Week 26 Day 3 |
| Project 6: Multi-Channel Notification Hub | Week 26 Day 4 |

All shipped. Phase 4 is complete.

---

## Self-Review Questions

**Queues and concurrency**
1. What three problems do queues solve that direct calls cannot?
2. What is the transactional outbox pattern and why does it matter?
3. When do you use BullMQ's built-in rate limiter vs a custom Redis counter?

**Microservices**
4. What are the four ways services can communicate?
5. When is a shared database across services acceptable?
6. What is a contract package and why do both services import it?

**Docker**
7. What is the difference between an image and a container?
8. Why use Alpine base images?
9. What does `depends_on: condition: service_healthy` do?

**Nginx**
10. Why terminate TLS at Nginx instead of in the Node app?
11. What security headers does every real site need?
12. How does Let's Encrypt issuance work?

**CI/CD**
13. What should CI run and what should it not run?
14. How do versioned image tags enable safe rollbacks?
15. Why use GitHub's built-in `GITHUB_TOKEN` instead of a PAT?

**Projects**
16. What is the `EXCLUDE USING gist` constraint and what does it protect against?
17. Why does the notification hub have two layers of queues (dispatch + channel)?
18. What does the attempt log let you answer that logs do not?

Target: 15/18. Anything less, revisit the relevant week.

---

## The Whole Curriculum So Far

Look at what you now have:

- A Kenyan e-commerce shop (Next.js + Postgres + M-Pesa + WhatsApp + Airtel + Stripe).
- A CRM (web + WhatsApp webhook + USSD).
- A WhatsApp bot managed from a state machine.
- A USSD menu system with i18n and Redis.
- A Telegram bot with group management.
- A chama savings platform combining all of the above.
- A published npm package for payments.
- A multi-service architecture with queues, Docker, and CI/CD.
- A booking system.
- A notification hub.

That is a startup-scale stack built from scratch in 26 weeks. The Capstone Sprint takes everything and makes it multi-tenant.

---

## Peer Coding Session

### Track A: End-to-end test
Write a single end-to-end test: spin up the full compose stack in a CI workflow, post an order, assert the WhatsApp hits a mock server, tear down. Green check = confidence.

### Track B: Image signing
Sign your Docker images with cosign. Verify signatures on the deploy server. This is supply chain security for the image registry.

### Track C: Rollback drill
Tag `v1.0.0`. Push a deliberate bug. Roll back to `v1.0.0`. Time the full rollback from "I notice the bug" to "the site is fixed". Target: under 5 minutes.

### Track D: Load test the notification hub
Fire 10,000 notifications in 60 seconds. Measure: dispatcher throughput, channel worker throughput, queue depth peak, failure rate. Publish the results.

---

## Weekend

The weekend polishes both projects and hardens the deploy pipeline. This is the last "real" weekend before the Capstone Sprint's very different shape.

Details in `week26/weekend/project.md`.

Phase 4 is done. The Capstone Sprint begins Monday -- four weeks to build The African SME OS.
