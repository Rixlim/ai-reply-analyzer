# AI Reply Analyzer

**Designing AI that reads the customer, but never decides for the salesperson.**

A mobile product design case study: when a customer replies, an assistant reads their real attitude, shows why, and drafts three replies. The salesperson can correct the AI and always sends the message themselves.

- **Role:** Product design (UX, UI, AI behaviour design) — Lim Win Hoong
- **Tools:** Figma (design + design system), Claude (AI rules + testing)
- **Status:** Designed screens and tested AI rules (6 cases plus one re-test, all pass). Not a live app.
- **Figma:** [AI Reply-Analyzer section](https://www.figma.com/design/xwnP4Mt56GIUFXmkbfHKVT/Product_P?node-id=1110-250)
- **Demo:** `demo/ai-reply-analyzer-prototype.mp4` (30-second clickable prototype)
- **Related:** [AI Sales Follow-up Assistant](https://github.com/Rixlim/ai-sales-followup), the first half of the same sales flow

---

## 1. Problem

The first project covers the customer who has **not** replied. This one covers the customer who **has**.

A reply is easy to misread. In my sales experience:

- "Thanks, looks good, we'll contact you if we need anything" sounds positive, but there is no date, no question and no next step. Most of these never come back.
- "Please message only, no calls" sounds neutral, but it means the customer wants distance. Calling again makes it worse.
- "We'll go back and think about it" often means the customer is comparing other providers. If they have already ordered elsewhere, chasing them wastes time.

A salesperson who misreads these either wastes follow-ups or pushes a customer who wants space. An AI that reads attitude can help, but a confident wrong guess is worse than no help. So the design question was **"how do we let AI read the customer without letting the salesperson trust a wrong read?"**

All example messages are rewritten from situations in my own sales work. They contain no real names or companies.

## 2. Scope: what I built and what I left out

| Feature | Decision | Why |
|---|---|---|
| Read the customer's attitude (5 types) | **Built** | Core value |
| Show the reason for the read | **Built** | Lets the salesperson check the AI |
| Three reply drafts (Safe / Proactive / Short) | **Built** | Gives a choice, not one answer |
| Salesperson can change the attitude | **Built** | AI will sometimes be wrong |
| Reply language (Auto, English, Bahasa Melayu, Chinese, Arabic) | **Built** | Different customer groups |
| "Not enough to tell" state with no drafts | **Built** | Refuse to guess on short replies |
| Auto-send replies | **Not built** | Same principle as project 1: the AI drafts, the human sends |
| Read WhatsApp or email directly | **Not built** | Needs integration. See next version |
| Remember each customer's conversation | **Not built** | Needs storage. See next version |

**Principle: the AI reads and drafts, the human decides and sends.**

## 3. What AI can get wrong, and how the design responds

| AI risk | Design response |
|---|---|
| Treats politeness as interest | Rule: "Interested" needs a concrete signal (date, question, offer request). Tone alone is not enough |
| Confident guess on a very short reply | "Not enough to tell" screen. It lists what to add and writes no drafts |
| Salesperson trusts a wrong read | "Why AI thinks so" quotes the exact words. A picker lets them change the attitude |
| Invented price, discount or deadline | Drafts use [PLEASE FILL IN] instead of making numbers up |
| Pushes a customer who wants space | "Keeping distance" replies are short and never suggest a call |
| Chases a customer who already chose someone else | "Comparing others" rule: no win-back unless the customer says they want to keep talking price |
| Reply in the wrong language | Language setting on the input page, and again on the result page |

## 4. Screens

| # | Screen | Purpose |
|---|---|---|
| 01 | Input | Paste the customer's reply, optional background, reply language, one button |
| 02 | Result | Customer's words, AI's read and confidence, why, attitude picker, language, three reply cards with Copy |
| 03 | Unsure | Shown when the AI cannot tell. Lists what is missing, no drafts, still lets the user pick an attitude |

![Input](assets/01-input.png) ![Result](assets/02-result.png) ![Unsure](assets/03-unsure.png)

**Design system:** the screens reuse the colours, spacing, radius, text styles and Button from project 1. Two new components were added to the design system: **Reply Card** and **Chip** (default and selected).

## 5. AI behaviour rules (prompt)

The full prompt is in [`prompt.txt`](prompt.txt). Key rules:

1. Being polite is not being interested. "Interested" needs a concrete signal.
2. If the reply is not enough, say "Not enough to tell", list what is missing, and write no drafts.
3. If the salesperson picked an attitude, use it and do not argue.
4. Never invent prices, discounts, deadlines or promises. Use [PLEASE FILL IN].
5. Each draft is under 60 words and sounds like a person.
6. Drafts follow the chosen language. Labels (ATTITUDE, WHY and so on) always stay in English. This rule comes from a problem in project 1, where English output sometimes drifted into Chinese.
7. Never suggest calling a customer who wants distance.
8. Output only the fixed format, with nothing added before or after. If there is no customer reply, the only missing item is "Customer reply".
9. Use [PLEASE FILL IN] only for a missing fact, never to mean "optional". With no background, do not refer to earlier messages or promise actions the input never mentioned.

| Attitude | Signals | Reply direction |
|---|---|---|
| Interested | Specific time, details, asks for a discount | Confirm the time, follow up on schedule |
| Polite decline | Friendly, but no date, no question | One short closing message, leave the door open, do not chase |
| Keeping distance | Several follow-ups unanswered, refuses calls | Short messages only, no calls |
| Price pushback | Keeps pushing the price down | Explain the value and how the price is built before moving on price |
| Comparing others | "Will go back and think", no question | Ask what they are comparing. No win-back if they already ordered elsewhere |

## 6. Testing

Six test cases, run in a Claude Project with `prompt.txt` as the project instructions. Each test was a new chat with three lines: customer reply, background, reply language. Screenshots are in `assets/`.

| Test | Input | Expected | Result |
|---|---|---|---|
| A. Interested | "Please call me next Tuesday morning. We're interested, the intro was good, but we'd like a discount." | Interested. Confirms Tuesday, does not invent a discount | **Pass.** Interested, High. Confirmed Tuesday morning and used [PLEASE FILL IN] for the discount. Minor: draft A promised to "check what we can offer", which the salesperson never said |
| B. Polite decline | "Noted, I'll reach out if I need anything." | Polite decline. Short closing reply, no chasing | **Pass with notes.** Polite decline, Medium, no chasing. Draft B assumed an earlier conversation ("what we covered") and used [PLEASE FILL IN: only if relevant] wrongly |
| C. Keeping distance | "No calls please, messages only. We're still considering and will let you know." Background: followed up several times, then called once | Keeping distance. Short replies, no call suggested | **Pass with notes.** Keeping distance, High. No call suggested. Draft B again offered a summary with the same wrong placeholder |
| D. Too short | "Noted, thanks." No background | Not enough to tell. Lists missing items, no drafts | **Pass.** No drafts. The missing items were a little generic but sensible |
| E. Language | Test B text, Reply Language set to Bahasa Melayu | Drafts in Bahasa Melayu, labels in English | **Pass.** Drafts in Bahasa Melayu, labels in English. Same placeholder issue in draft B. Not checked by a native speaker |
| F. No reply at all | The prompt with an empty customer reply | Not enough to tell. MISSING lists only "Customer reply". Nothing outside the format | **Pass after a fix.** Before the fix it listed the input fields and added a Chinese note outside the format (`test-f-before.png`). After rules 8 and 9: only "Customer reply" (`test-f.png`) |
| G. Re-test of B on the updated prompt | Same as B, with rules 10 and 11 added | No "what we covered", no wrong placeholder | **Pass.** Polite decline, Medium. No reference to an earlier conversation and no wrong placeholder (`test-b-v2.png`). Draft B still offers to send details, but only "if it helps later" |

### What changed because of testing and review

- **Rules 8 and 9 (from test F):** the first run listed the input fields as "missing" and added a Chinese note outside the format.
- **Rules 10 and 11 (from tests B, C and E):** the same wrong placeholder appeared in three of six tests, and the AI referred to a conversation that the input never mentioned. These are small ways of inventing facts, so I added rules against both.
- **Design changes after review:** added a picker so the user can change the attitude; kept "Why AI thinks so" so the read can be checked; gave the "Not enough to tell" screen the result-page header so it reads as a result; added the reply language setting.

## 7. Known issues and limitations

- **Copy and paste for every reply is slow.** For a salesperson with dozens of customers it is a real cost. This prototype accepts that to test the idea first. See next version.
- Only 6 test cases and one re-test. Not enough to claim reliability.
- No user testing yet. Whether the reads feel right is my judgement, not a salesperson's.
- "Price pushback" and "Comparing others" overlap when a customer says another provider is cheaper. This was not tested.
- Arabic is shown in a left-to-right layout. A proper right-to-left layout is not designed.
- Bahasa Melayu drafts have not been reviewed by a native speaker. Arabic and Chinese drafts were not tested.
- The prototype is not connected to a live AI.

## 8. Next version

- **Less copying:** read the clipboard on open, then connect to WhatsApp or email so a new reply is analysed automatically.
- **One thread per customer:** the AI remembers earlier messages, so Background is no longer typed by hand.
- **Right-to-left layout** for Arabic.
- **Connect the form to an AI API** and run a working build.
- **User testing:** 5 salespeople, measuring how often the AI's read is changed and how often a draft is sent unedited.
- **Combine with project 1:** one flow, with no reply going to the follow-up draft and a reply going to the analyzer.

## Files

```
README.md
prompt.txt    AI rules (section 5)
demo/         prototype screen recording
assets/       design screens (01-input, 02-result, 03-unsure)
              test results (test-a to test-f, test-f-before, test-b-v2)
```
