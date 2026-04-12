# Week 30, Day 5: Demo Day

Ship it. Thirty weeks of build. Today is the last class.

**Estimated time:** as long as it takes.

---

## Morning Checklist

Two hours before the demo:

- [ ] Sleep.
- [ ] Eat.
- [ ] Staging is up; run the demo once to verify.
- [ ] Production is up; run a no-op sanity check (`curl https://smeos.co.ke/api/health`).
- [ ] Reset the demo database if your demo uses staging.
- [ ] Charge all devices.
- [ ] Test network at the venue.
- [ ] Backup video is on a second device.
- [ ] Demo one-pager is printed.
- [ ] Your own notes: open and ready.

---

## The Demo

Do it exactly as rehearsed. If something breaks, use the backup. Do not apologise. Do not explain why things went wrong. Push through.

Time it. 5 minutes maximum.

---

## Q&A

Expect these questions. Prep the answers.

**Q: Did you use AI for this?**
A: "Where it was useful. I wrote the architecture decisions and the data model myself. AI helped me type faster. The hard parts -- the RLS policies, the idempotency in the payments flow, the queue strategy -- came from the coursework."

**Q: How long did this take?**
A: "Four weeks for the Capstone, 26 weeks of foundation. The week breakdown is in the README."

**Q: Is this in production with real users?**
A: "It is deployed on a real server with real domains, TLS, and monitoring. No paying customers yet -- that is what comes after graduation."

**Q: What is next?**
A: "[Your real answer. Be specific. 'Find three Kenyan businesses to onboard in the next month' is a good answer. 'Make money' is a bad answer.]"

**Q: What was hardest?**
A: "[Real answer. The idempotency work in Week 18 was a big one for me. Or the RLS debugging. Or the deploy pipeline. Pick one and be honest.]"

**Q: What would you change if you did it again?**
A: "[Real answer. Not 'nothing, it is perfect'. Something like 'I would have picked one payment provider and shipped that first, instead of trying to unify three from day one'.]"

---

## After The Demo

Whatever happens, you finished it. That is the achievement. The grade does not matter as much as the fact that you built a multi-service, multi-tenant, multi-channel platform from scratch in 30 weeks.

Back up your code. Push everything. Tag the final version.

```bash
git tag v1.0.0-final
git push --tags
```

Write a one-paragraph post-mortem. What surprised you. What you got wrong. What you are proud of. Save it somewhere you will read in a year.

---

## The Thirty-Week Curriculum, Looking Back

You learned:

- Phase 1 (Weeks 1-6): Web fundamentals, JS, React basics.
- Phase 2 (Weeks 7-13): React deep, Postgres, auth, WhatsApp, USSD, clean architecture.
- Phase 3 (Weeks 14-22): Next.js, Stripe, Airtel, M-Pesa, webhook security, Telegram, cron, chamas.
- Phase 4 (Weeks 23-26): BullMQ, microservices, Docker, Nginx, CI/CD.
- Phase 5 (Weeks 27-30): Multi-tenant platform from scratch.

Every week built on the last. Every project reused earlier code. By Week 30 you were not learning new tech -- you were applying 26 weeks of tools to a real product.

That is engineering. Not memorising, not copy-pasting, not cargo-culting. Building.

---

## What To Do Next

In the 30 days after graduation:

1. **Keep the project live.** Pay for the server. Bring back the demo if anyone asks.
2. **Get 3 real users.** Onboard three actual small businesses. Listen to what they need.
3. **Contribute to an open source project.** Pick one thing in the Node or Postgres ecosystem. Fix one bug.
4. **Write about what you built.** One blog post, one thread, one LinkedIn post. Own the narrative.

In the 90 days after:

5. **Decide what this becomes.** A product you charge for? A portfolio piece? A job reference? All three are valid. Pick one and commit.
6. **Find a mentor.** Someone who has shipped a multi-tenant SaaS before. Buy them coffee. Ask one specific question.
7. **Apply what you built.** Your next job interview: do not mention "a course". Show the repo. Walk through the RLS policies. Show the isolation test. That is how you get a Nairobi backend job.

---

## Graduation

Thirty weeks ago you did not know how JavaScript promises work. Today you built an African SME OS with row-level security, queue-backed notifications, and a published payments package. That is not nothing.

Back up your work. Shake hands. Take a day off.

Then come back and ship something.

Welcome to the trade.
