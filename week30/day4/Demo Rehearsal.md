# Week 30, Day 4: Demo Rehearsal

By the end of today, your demo is rehearsed three times, the script is polished, the backup plan if something fails is documented, and you know exactly what you are going to say for every click.

**Estimated time:** 3 hours.

---

## The 5-Minute Demo

Rehearse this exactly three times today. Time yourself. Under 5 minutes.

### The Hook (30 seconds)

"Small businesses in Kenya run on three things: M-Pesa, WhatsApp, and USSD. Each of those is its own silo. My project is called SME OS. It gives any small business one URL where they sign up, and within seconds they have a working USSD code, a WhatsApp bot, and a wallet. Let me show you."

### The Signup (30 seconds)

- Go to `smeos.co.ke/signup`.
- Type "Kilimani Kitchen" / "kilimani@test.co.ke" / password.
- Click Submit.
- Land on the dashboard.

"Thirty seconds. Kilimani Kitchen is now a tenant. They have a wallet, a USSD code, and a WhatsApp bot. Everything is empty but it is real."

### The Product Setup (30 seconds)

- Click Products.
- Click Add product.
- "Ugali and Nyama -- KSh 400 -- stock 20".
- Submit.

"One product. Ready to sell."

### The Customer Flow (2 minutes)

- Pull out your real phone.
- Dial `*384*1234*1#`.
- Show the phone screen. "This is Brian, a customer. He does not have a smartphone. He dials the code."
- Navigate: "1. Browse. 1. Ugali and Nyama. 1. Buy now."
- STK prompt. "My phone is getting the prompt now."
- Enter PIN.
- Wait.
- "Confirmation on screen. SMS incoming."
- WhatsApp message arrives with confirmation.

"One USSD call. Payment via M-Pesa. Confirmation via WhatsApp. Brian did not download an app, did not visit a website."

### The Dashboard View (1 minute)

- Switch to the laptop.
- Refresh the dashboard.
- Overview: 1 order today, KSh 400 revenue, KSh 400 wallet balance.
- Click Orders. Show the new row.
- Click Inbox. Show the WhatsApp message.
- Click Wallet. Show the credit transaction.

"The tenant sees everything in one place. USSD order, WhatsApp confirmation, wallet credit. One dashboard."

### The Isolation Moment (30 seconds)

- Open a second browser tab with Tenant B's dashboard.
- "This is Westlands Gym. Different tenant."
- Overview: 0 orders.
- "No data from Kilimani Kitchen. Not because the code filters, but because Postgres row-level security makes it physically impossible for one tenant to see another's rows."

### The Close (30 seconds)

"This was built in the final 4 weeks of a 30-week program. Before this the course covered Postgres, Next.js, M-Pesa, WhatsApp, USSD, Telegram bots, BullMQ, Docker, CI/CD. Every one of those pieces is here. This is not a prototype. It runs on a $6/month server, has real TLS, real backups, real monitoring. Thank you."

---

## Things That Will Go Wrong

Rehearse the backup plan for each.

### USSD simulator does not respond

Have a screenshot ready. Walk through the flow with "this is normally what you would see" voiceover.

### STK prompt does not arrive

Sandbox can lag. Have an SMS ready to send that says "Simulating STK prompt". Or pre-record the phone screen video and play that.

### WhatsApp message does not arrive

Meta webhooks can lag. Have a screenshot. Or deliberately pre-load a confirmation in the test number.

### Dashboard does not update

Refresh. Explain that in production this would push instantly via websockets, but for the Capstone it is poll-based.

### Site is down

Have the local version running as a backup. Switch to `localhost:3000` in the browser.

### Everything works

Smile. Pause. Let the room see the magic moment.

---

## Rehearsal Protocol

Three rehearsals today. Each one:

1. Reset the database. Fresh state.
2. Start the timer.
3. Run the demo start to finish without looking at notes.
4. Stop the timer.
5. Write down what felt slow, what tripped you, what you forgot.

Target rehearsal times:
- First run: ~7 minutes. Too slow. Cut 2 minutes.
- Second run: ~5 minutes. Close enough.
- Third run: ~4:30. Smooth.

Record the third run. Watch it back. Adjust one last time.

---

## The Materials

Have these on hand:

- Laptop with the dashboard open and staging version loaded.
- A second laptop or phone showing the USSD simulator.
- A real Kenyan phone for the STK prompt moment.
- A backup recorded video of the demo for if live fails.
- A printed one-page architecture diagram to hand out.
- Screenshots of every key screen in a shared folder.

---

## What To Say And Not Say

**Say:**
- "Built in 4 weeks on top of a 26-week foundation."
- "Multi-tenant with Postgres row-level security."
- "Every cent is accounted for in a transactional wallet."
- "Queue-based so channel failures do not cascade."

**Do not say:**
- "I do not know why that is happening."
- "I could have done this better if I had more time."
- "This is just a prototype."
- "I have been up all night."

Own what you built.

---

## Checkpoint

1. Three rehearsals completed today.
2. Third run under 5 minutes.
3. Backup plan for each failure mode.
4. A recording you watched back at least once.

Commit:

```bash
git add docs/demo-script.md
git commit -m "docs: demo day rehearsal script"
```

---

## What You Learned

- A rehearsed demo looks magic. An unrehearsed one looks like a tutorial.
- Have a backup for every failure mode.
- The close matters as much as the hook.

Tomorrow: demo day.
