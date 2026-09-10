# Build with AI Bootcamp — Hands-on Prompt Pack

Everything in a fenced block is paste-ready. Replace only the `[bracketed]` parts.

**How to use this in the session**

1. Pick **one track** from Section 1 and carry it through every lab. Don't switch halfway.
2. Warm up your prompting in the AI Studio playground (Section 2).
3. Design it in Stitch (Section 3), build and deploy it in AI Studio (Section 4).
4. Watch the Antigravity demo, then run Section 5 on your own repo tonight.

Section 6 is stretch material for anyone who finishes early. Section 7 is the facilitator crib sheet.

---

## 1. Pick your track

Eight app ideas. Each one has a genuinely different **AI-shaped job** at its centre — so if the room spreads out, you collectively demo most of the Gemini family. Pick the one closest to a problem you actually have.

| # | Track | The problem | What Gemini does | Capability on show |
|---|---|---|---|---|
| 1 | **TiffinTrack** | You cook at home and have no idea what it costs you | Turns "two rotis, dal, half a cup of rice" into a normalised dish, category and ingredient cost | Structured extraction + estimation |
| 2 | **SkillGap** | You're applying to jobs and guessing what's missing | Compares your resume against a job description and scores the gaps | Long-context comparison + scoring |
| 3 | **PrepDeck** | You have notes but no time to make revision material | Turns a page of notes into flashcards and a five-question quiz | Generation from source material |
| 4 | **ClinicQueue** | A small clinic's front desk triages walk-ins by gut feel | Reads the complaint and routes to the right queue with an urgency flag | Classification + routing |
| 5 | **FieldNote** | Site inspections end as a phone full of unlabelled photos | Reads a site photo and drafts the observation, defect type and severity | Image understanding |
| 6 | **KiranaBooks** | Small-shop credit ledgers live in a paper notebook | Reads a photo of a handwritten ledger page and extracts the entries | Multimodal extraction |
| 7 | **ScriptRoom** | Creators burn hours on hooks and captions | Generates platform-specific variants inside a fixed brand voice | Constrained creative generation |
| 8 | **SocietyDesk** | Housing-society complaints arrive as angry WhatsApp messages | Classifies the complaint, strips the heat, and drafts a calm reply for a human to approve | Classification + tone-controlled drafting |

> **Choosing well:** the best track for you is the one where you can already picture the screen you'd stare at. If two look equally good, take the one whose data you could fake in thirty seconds.

---

## 2. Warm-ups — AI Studio playground

Ten minutes, six exercises. Open `aistudio.google.com`, pick **gemini-3.8-flash**, and run these in order. The point isn't the output — it's watching what each addition changes.

### W1 — The vague/specific test

Run both. Time how long you'd spend fixing each result.

**A (vague):**

```
Write a product description for my app.
```

**B (specific):**

```
You are a copywriter for developer tools.

Write a Play Store short description for TiffinTrack, an app that logs
home-cooked meals and their ingredient cost.

Audience: working professionals in Indian metros who cook most days.
Tone: plain, warm, zero hype. No exclamation marks. No "revolutionise",
no "seamless", no "game-changer".
Length: 78 characters maximum.
Only reference these features: meal log, monthly spend, shared household budget.

Return 3 options as a JSON array of strings and nothing else.
```

**Notice:** B is longer but took forty seconds to write. A costs ten minutes of editing.

---

### W2 — Build a prompt in six moves

Same task, one layer at a time. Run each and watch the output tighten.

**Move 1 — task only**

```
Classify this support ticket.

Ticket: "Ordered a phone case on the 3rd, payment went through twice,
only one order shows in my account. I've called twice, nobody calls back."
```

**Move 2 — add the role**

```
You are a support triage analyst for an e-commerce company.

Classify this support ticket.

Ticket: "Ordered a phone case on the 3rd, payment went through twice,
only one order shows in my account. I've called twice, nobody calls back."
```

**Move 3 — add context (the actual queues)**

```
You are a support triage analyst for an e-commerce company.

Route this ticket to exactly one queue:
- payments — duplicate charges, refunds, failed transactions
- orders — missing items, wrong item, delivery delays
- account — login, address, profile changes
- returns — return requests and pickup scheduling
- escalation — anything where the customer has already been failed twice

Ticket: "Ordered a phone case on the 3rd, payment went through twice,
only one order shows in my account. I've called twice, nobody calls back."
```

**Move 4 — add constraints**

```
[...same as Move 3, plus:]

Rules:
- Choose exactly one queue. Never two.
- Never invent an order ID, amount or date the customer didn't state.
- If the ticket mentions repeated failed contact, escalation wins over the
  topical queue.
- If you genuinely cannot tell, use "account" and mark confidence low.
```

**Move 5 — add output format**

```
[...same as Move 4, plus:]

Return only JSON, no markdown fences, no prose:
{ "queue": string, "priority": "P1" | "P2" | "P3", "confidence": "high" | "medium" | "low", "reason": string (max 15 words) }
```

**Move 6 — add examples**

```
[...same as Move 5, plus:]

Examples:

Ticket: "Where is my order, it said Tuesday and it's Friday."
{ "queue": "orders", "priority": "P2", "confidence": "high", "reason": "Delivery delay, no prior contact" }

Ticket: "Charged 4,499 twice. Raised this last week and again on Monday. Still nothing."
{ "queue": "escalation", "priority": "P1", "confidence": "high", "reason": "Duplicate charge plus repeated failed contact" }
```

**Notice:** the answer stops changing between runs somewhere around Move 4. That's the point where it becomes something you could ship.

---

### W3 — Force structured output

Turn on **Structured output** in the right panel, or paste this and see the difference.

```
Extract every dish from this meal description.

For each dish return: name, category (breakfast/lunch/dinner/snack/beverage),
estimated ingredient cost in INR for one serving, and confidence.

Rules:
- Ingredient cost only, not restaurant price.
- If the description is too vague to estimate, set cost to null and
  confidence to "low".
- Never invent a quantity the user didn't give.
- Return only a JSON array. No markdown fences, no explanation.

Description: "had two rotis with dal and half a bowl of rice for lunch,
plus chai in the evening"
```

**Notice:** ask for JSON in prose and you'll get JSON *most* of the time. Pass an actual schema and you get it every time. That difference is the whole reason this is production-viable.

---

### W4 — Few-shot beats instructions

Try to describe "urgent" in words. Then try showing it.

```
Label each message as URGENT or ROUTINE.

Examples:
"Lift stuck between floors, someone is inside" -> URGENT
"Lift making a noise for the last few days" -> ROUTINE
"Water tank overflowing into the parking" -> URGENT
"Requesting a second water tank next quarter" -> ROUTINE
"Smell of gas near B wing stairs" -> URGENT
"Streetlight near gate 2 is out" -> ROUTINE

Now label these:
1. "Sparking from the meter box on 3rd floor"
2. "Painting of the compound wall is peeling"
3. "Security guard didn't show up for the night shift"
4. "Kids playing cricket in the parking again"
```

**Notice:** six examples encoded a judgement call ("is anyone in danger right now?") that would take three paragraphs to write badly.

---

### W5 — Grounding

Turn on the **Google Search** tool, then run:

```
What are the current free-tier rate limits for gemini-3.8-flash on the
Gemini API? Give requests per minute and per day.

Cite the official Google documentation page you used. If you cannot find
an official Google page stating this, say so plainly instead of estimating.
```

**Notice:** run it once with Search off and once with it on. The confident wrong answer is the lesson.

---

### W6 — System instruction persistence

Put this in the **System instructions** box:

```
You are a triage assistant for a housing society office in Pune.

Always:
- Reply in under 40 words.
- Stay factual and calm even if the message is angry.
- End with the queue name in square brackets, e.g. [plumbing].

Never:
- Promise a timeline.
- Apologise on behalf of the committee.
- Use exclamation marks.
```

Then send three unrelated messages in a row and watch the rules hold without repeating them.

---

## 3. Lab 1 — Stitch

`stitch.withgoogle.com` · 15 minutes · **Deliverable: three screens you'd happily show someone, plus the code export.**

### 3.1 The master template

```
Design a [mobile / web] app called [NAME] for [specific audience] who [specific situation].

Screens:
1. [screen name] — [what's on it, what the user does there]
2. [screen name] — [what's on it]
3. [screen name] — [what's on it]

Style: [2–3 concrete adjectives], [colour direction], [shape and spacing direction].
Avoid: [what you don't want].

Include: [component 1], [component 2], and a [empty state / error state].

[Platform detail], light and dark theme.
```

**The three words that waste your credits:** "modern", "clean", "professional". They describe nothing. Say "high contrast, one accent colour, no illustrations" instead.

### 3.2 Ready-made track prompts

**1 · TiffinTrack**

```
Design a mobile app called TiffinTrack for working professionals in Indian
metros who cook most of their meals at home and want to know what it costs.

Screens:
1. Home — today's meals as cards, each with a cost chip and time of day,
   plus a running total for the day at the top
2. Add meal — a single text field where you describe the meal in plain
   language, a date/time picker, and a save button
3. Monthly spend — a simple bar chart by week, a total, and a breakdown
   by meal category

Style: calm and high contrast, generous whitespace, one warm accent colour,
rounded cards, large readable numbers.
Avoid: illustrations, gradients, decorative icons.

Include: a bottom nav with three tabs, a cost chip on every meal card, and
a friendly empty state on the home screen for a day with no meals logged.

Android-first, light and dark theme.
```

**2 · SkillGap**

```
Design a web app called SkillGap for mid-career engineers in India applying
to jobs, who want to know what's missing from their resume before they apply.

Screens:
1. Upload — two side-by-side drop zones, one for a resume PDF and one for a
   pasted job description, with a prominent Analyse button
2. Results — an overall match score as a large number, then three grouped
   lists: strong matches, partial matches, and missing entirely
3. Action plan — an ordered checklist of what to add or learn, each item with
   an effort estimate

Style: dense and functional, near-black text on white, one blue accent,
sharp corners, data-table feel.
Avoid: hero images, marketing copy, rounded pill buttons.

Include: a sticky results header showing the job title, colour-coded gap
severity, and a loading state for the analysis step.

Desktop-first, light theme only.
```

**3 · PrepDeck**

```
Design a mobile app called PrepDeck for students revising from their own
class notes the night before an exam.

Screens:
1. Import — paste or photograph notes, choose a subject tag, generate
2. Flashcards — one card at a time, tap to flip, swipe for know/don't know,
   with a progress bar
3. Quiz results — score, the questions you got wrong, and a "revise these"
   button

Style: focused and low-distraction, dark theme first, one bright accent for
progress, large type, minimal chrome.
Avoid: badges, streaks, confetti, anything gamified.

Include: a card counter, a swipe hint on the first card, and an empty state
for a subject with no cards yet.

Android-first, dark and light theme.
```

**4 · ClinicQueue**

```
Design a tablet app called ClinicQueue for the front desk of a small
two-doctor clinic in a tier-2 Indian city.

Screens:
1. Walk-in intake — patient name, age, phone, and a large free-text box for
   "what's the problem", with a big Add to queue button
2. Live queue — a list of waiting patients with token number, wait time,
   assigned doctor, and an urgency indicator
3. Patient detail — the intake text, the suggested queue and urgency, and
   two buttons: accept or reassign

Style: high contrast for a bright reception desk, very large touch targets,
one accent colour plus red for urgent only, no small text anywhere.
Avoid: thin fonts, subtle greys, hover-dependent controls.

Include: a token number badge on every row, a clearly separated urgent
section at the top, and an empty state for an empty queue.

Tablet landscape, light theme only.
```

**5 · FieldNote**

```
Design a mobile app called FieldNote for civil site engineers doing
walk-through inspections on construction sites.

Screens:
1. Capture — camera view with a shutter button, a location chip, and a
   one-tap "add note" affordance
2. Observation — the captured photo at the top, an auto-drafted observation
   you can edit, a defect-type selector, and a severity picker
3. Report — the day's observations grouped by severity, with a share button

Style: rugged and legible in sunlight, very high contrast, thick touch
targets, one safety-orange accent, no subtle shadows.
Avoid: pastel colours, thin outlines, small icons.

Include: a photo thumbnail strip, a severity colour key, and an empty state
for a site with no observations yet.

Android-first, light theme only.
```

**6 · KiranaBooks**

```
Design a mobile app called KiranaBooks for small neighbourhood shop owners
who currently track customer credit in a paper notebook.

Screens:
1. Customers — a searchable list, each row showing name and outstanding
   balance, with the highest balances at the top
2. Scan ledger — camera view aimed at a notebook page, with a capture button
   and a short instruction line
3. Review entries — the extracted rows in an editable table (date, item,
   amount), with a confirm-all button

Style: simple and trustworthy, very large numbers, one green accent for paid
and one red for outstanding, high contrast, Devanagari-friendly type sizing.
Avoid: dense tables, small fonts, English-only labels in the UI chrome.

Include: an outstanding-total banner, a per-row confidence indicator on the
review screen, and an empty state for a shop with no customers yet.

Android-first, light theme only.
```

**7 · ScriptRoom**

```
Design a web app called ScriptRoom for solo creators who publish short-form
video across YouTube Shorts, Reels and LinkedIn.

Screens:
1. Brief — a topic field, a platform multi-select, a tone selector, and a
   brand-voice text area that persists between sessions
2. Variants — generated hooks and captions in a three-column board, one
   column per platform, each card copyable and regeneratable
3. Calendar — a week view with cards dragged onto days

Style: creative-tool feel without being loud, near-black canvas, one electric
accent, card-based, tight spacing.
Avoid: stock illustrations, gradient buttons, emoji in the UI chrome.

Include: a per-card character counter, a regenerate icon on every card, and
an empty state for a week with nothing scheduled.

Desktop-first, dark theme first.
```

**8 · SocietyDesk**

```
Design a web app called SocietyDesk for the office secretary of a housing
society who receives complaints over WhatsApp and has to reply to all of them.

Screens:
1. Inbox — incoming complaints as rows with sender, first line, category and
   urgency, urgent ones grouped at the top
2. Complaint detail — the original message, the suggested category, and a
   drafted reply in an editable box with Approve and Rewrite buttons
3. Resolved — a filterable log of what was handled, by whom and when

Style: calm and administrative, plenty of whitespace, one muted blue accent,
red used only for urgent, comfortable reading width.
Avoid: dashboard widgets, charts, anything that looks like analytics.

Include: an explicit "nothing is sent without your approval" line above the
draft, a category chip on every row, and an empty state for a clear inbox.

Desktop-first, light theme only.
```

### 3.3 Refinement prompts (annotate or chat)

Use these **one at a time**. Two changes in one round and you can't tell which one helped.

```
Increase the contrast between the card background and the page background.
```

```
The spacing is uneven — make all vertical gaps between cards identical.
```

```
Replace the icons with plain text labels. I want to see if the layout still reads.
```

```
Make the primary number on this screen twice the size of everything else.
```

```
Redesign this screen assuming the user has exactly one item, not twelve.
```

```
Same layout, but a single accent colour instead of three.
```

```
Show me the dark theme version of this screen only.
```

### 3.4 Theme variants worth trying

Run your track prompt again with only the Style block swapped:

```
Style: editorial and typographic, serif headings, off-white background,
one deep accent, wide margins, no cards at all.
```

```
Style: dense and utilitarian like an airline ops console, monospace numbers,
tight rows, minimal colour, information over comfort.
```

```
Style: soft and reassuring, muted palette, very rounded corners, generous
padding, one gentle accent, nothing sharp.
```

---

## 4. Lab 2 + 3 — Google AI Studio

`aistudio.google.com` · 20 min build + 15 min deploy · **Deliverable: two projects live on a URL.**

### 4.1 The handoff prompt (Build mode, first message)

```
I'm giving you a UI design exported from Google Stitch, plus its DESIGN.md.

Build this as a working app.

Rules:
- Keep the layout, spacing, type scale and colours exactly as designed.
- Make the screens actually navigate and the forms actually submit.
- Use local state and placeholder data. No backend yet.
- Do not add screens, features, branding or a landing page I didn't ask for.
- Do not add a library unless you tell me why first.

--- DESIGN.md ---
[paste DESIGN.md here]

--- CODE ---
[paste the Stitch code export here]
```

### 4.2 The "now add the intelligence" prompt

This is the step most people skip, and it's the whole reason you're here. Send it as a **separate second message**, after the UI is working.

**Generic shape:**

```
Now add one real Gemini call.

When the user [does this specific thing], call the Gemini API with
[this input] and use the result to [do this specific thing in the UI].

Use model gemini-3.8-flash with a response schema so the output is typed
JSON, not prose. Show a loading state while it runs and a readable error
state if it fails.

Keep the API key server-side. Do not put it in client code.
```

**Filled per track:**

| Track | The one AI call |
|---|---|
| TiffinTrack | When the user saves a meal description, extract dish name, category, estimated ingredient cost and confidence, and render it on the meal card |
| SkillGap | When both inputs are present, compare resume to job description and return a match score plus three grouped gap lists |
| PrepDeck | When the user imports notes, generate 10 flashcards and 5 multiple-choice questions with answers and one-line explanations |
| ClinicQueue | When intake is submitted, classify the complaint into a queue with an urgency level and a one-line reason for the front desk |
| FieldNote | When a photo is captured, send the image and return a drafted observation, defect type and severity |
| KiranaBooks | When a ledger page is photographed, extract every row as date, item and amount, each with a confidence value |
| ScriptRoom | When the brief is submitted, generate 3 hooks and 3 captions per selected platform, inside the brand-voice text the user saved |
| SocietyDesk | When a complaint is opened, return a category, an urgency, and a calm drafted reply — always shown for approval, never auto-sent |

### 4.3 System instructions and schemas

Paste the system instruction into the model settings, not into every user turn.

**TiffinTrack**

```
You are a meal-cost estimator for a home-cooking expense tracker used in
Indian metros.

Given a free-text meal description, return one entry per distinct dish.

Rules:
- Estimate INGREDIENT cost for one serving in INR. Never a restaurant price.
- Use typical urban Indian grocery prices as your baseline.
- Never invent a quantity the user did not state.
- If a dish is too vague to estimate, set cost to null and confidence "low".
- Return only JSON matching the schema. No prose. No markdown fences.
```

```json
{
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "dish": { "type": "string" },
      "category": { "type": "string", "enum": ["breakfast","lunch","dinner","snack","beverage"] },
      "cost_inr": { "type": ["number","null"] },
      "confidence": { "type": "string", "enum": ["high","medium","low"] }
    },
    "required": ["dish","category","cost_inr","confidence"]
  }
}
```

**ClinicQueue**

```
You are a front-desk triage assistant for a small two-doctor clinic.
You do NOT diagnose. You route.

Given a patient's description of their problem, choose one queue:
general, paediatric, dressing_or_injection, follow_up, or urgent_referral.

Rules:
- Chest pain, breathlessness, uncontrolled bleeding, head injury, seizure,
  or anything the patient calls sudden and severe goes to urgent_referral.
- Never name a condition or suggest a treatment.
- Reason must be under 12 words and must quote or paraphrase only what the
  patient said.
- Return only JSON matching the schema.
```

```json
{
  "type": "object",
  "properties": {
    "queue": { "type": "string", "enum": ["general","paediatric","dressing_or_injection","follow_up","urgent_referral"] },
    "urgency": { "type": "string", "enum": ["routine","soon","immediate"] },
    "reason": { "type": "string" }
  },
  "required": ["queue","urgency","reason"]
}
```

**SocietyDesk**

```
You are a drafting assistant for a housing society office. A human approves
every message before it is sent — you never send anything.

Given a complaint message, return a category, an urgency, and a drafted reply.

Draft rules:
- Under 60 words.
- Acknowledge the specific issue named. Do not generalise it.
- Never promise a date, a cost, or an outcome.
- Never apologise on behalf of the committee or admit fault.
- Match the resident's language: if they wrote in Hindi or Gujarati, reply in
  the same language, in the same script.
- Stay calm regardless of the tone of the complaint.

Return only JSON matching the schema.
```

**Anywhere else:** keep the shape — a role, the closed list of allowed outputs, the "never" rules, and "return only JSON matching the schema."

### 4.4 Iteration prompts

One fix per message. Batching makes failures untraceable.

```
The [specific element] on the [specific screen] is [specific problem].
Fix only that. Change nothing else.
```

```
Here is the exact error from the console:

[paste the full error]

I expected [what should have happened]. Fix the cause, not the symptom.
```

```
The Gemini call works but the response sometimes comes back as a string
instead of an object. Enforce the schema and handle the failure case.
```

```
Add a loading state to the [X] action and a visible error state if the
API call fails. Don't change the layout.
```

```
Show me exactly where the API key is referenced in this project.
```

### 4.5 Deploy

```
Prepare this for deployment: move the Gemini call behind a server route so
the key is never in client code, add a basic rate guard, and tell me what
environment variables I need to set.
```

Then hit deploy, open the URL **on your phone**, and paste it in the chat. Someone else using it is the only proof that counts.

---

## 5. Antigravity — take-home prompts

`antigravity.google` · Demo in the room, run these on your own repo tonight.

### 5.1 The task template

```
Goal: [one sentence describing the outcome, not the method]

Context:
- This repo is [what it does, in one line]
- The relevant code is in [paths]
- [Anything non-obvious about how it's wired]

Done means:
- [observable check 1 — something you could see in a browser or a test run]
- [observable check 2]
- The app still builds and existing tests still pass

Constraints:
- Don't change [X]
- Don't add a dependency without telling me first
- Don't reformat files you didn't need to touch

Show me the plan before you write any code. I'll approve it.
```

### 5.2 Five concrete tasks

**Add a feature**

```
Goal: Add CSV export to the monthly spend screen.

Done means:
- A visible Export button on that screen only
- Clicking it downloads a .csv with one row per meal: date, dish, category, cost
- Empty state exports a file with just the header row, not a crash

Constraints: no new dependencies, keep the existing button styling.

Plan first.
```

**Fix a bug with a reproduction**

```
Goal: Fix the duplicate-entry bug on the add-meal screen.

Reproduce: add a meal, tap save twice quickly, look at the home screen.
Two identical cards appear.

Done means: saving twice fast produces one entry, and you show me a browser
recording proving it.

Find the actual cause. Don't just disable the button.
```

**Write tests around behaviour you're afraid of**

```
Goal: Write tests for the Gemini response handling.

Cover: valid JSON, JSON with an unexpected extra field, a truncated response,
a plain-prose response, a timeout, and a 429.

Done means: all six cases have a test, they pass, and none of them hit the
real API.
```

**Refactor with a boundary**

```
Goal: Extract all Gemini API calls into a single module.

Done means:
- Exactly one file imports the SDK
- Every call site uses the new module
- Behaviour is identical — prove it by running the existing tests before
  and after and showing me both runs

Don't change any prompt text or model name while doing this.
```

**Browser-verify something you don't believe**

```
Open the running app in the browser. Walk through this flow:
add a meal, check the home screen, open monthly spend, switch to dark theme.

Screenshot each step. If anything doesn't match what the code claims should
happen, tell me which step and fix it, then show me again.
```

### 5.3 Prompts to use mid-run

```
Stop. Your plan assumes [X]. That's wrong because [Y]. Revise the plan
before continuing.
```

```
You said it works. Show me the artifact that proves it.
```

```
That's more change than I asked for. Revert everything except [the specific
thing] and show me the diff.
```

---

## 6. Stretch variants

For anyone who finishes early, and for the second half of your week.

### 6.1 Multimodal — send an image

Works in the playground immediately. Attach a photo, then:

```
This is a photo of a construction site element.

Return:
- observation: what you can actually see, in one sentence, factual only
- defect_type: one of crack, spalling, honeycombing, seepage, alignment,
  reinforcement_exposure, none_visible
- severity: low, medium, high
- confidence: high, medium, low

Rules:
- Describe only what is visible. Do not infer causes.
- If the photo is too dark, blurry or close-cropped to judge, say so and set
  confidence to low.
- Return only JSON.
```

Swap the domain for your track: a handwritten ledger page, a whiteboard of notes, a lab report, a food plate.

### 6.2 Grounding with Google Search

```
[Your question]

Ground your answer in Google Search. Cite the specific pages you used.
If the sources disagree, say so and show both. If you cannot find a primary
source, say so rather than filling the gap.
```

### 6.3 Image generation — Nano Banana

Use `gemini-3.1-flash-image` for the workhorse, `gemini-3-pro-image` when there's text in the image.

```
Generate an empty-state illustration for a meal-logging app.

Subject: a single empty steel tiffin box, lid ajar, seen from a three-quarter
angle.
Style: flat vector, two colours only — warm terracotta and off-white, no
gradients, no outlines, no shadow.
Composition: centred, generous negative space, square, transparent background.
No text.
```

```
Generate an app icon for a housing-society complaints tool.

Subject: a simple building silhouette with a speech mark cut out of it.
Style: solid single colour on a soft neutral background, geometric, no
gradient, no bevel, no 3D.
Format: square, safe margins for rounded-corner masking.
```

### 6.4 Voice — Live API

Try in the playground's Stream / Live tab:

```
You are a hands-free meal logger. The user is standing in a kitchen and
cannot type.

When they describe a meal, confirm what you heard in under 10 words, then
ask only for what's genuinely missing.

Never ask more than one question at a time.
Never read numbers back as digits — say them as words.
If they say "done", summarise the day's meals and stop.
```

### 6.5 Chain two calls instead of one big prompt

```
Call 1 — extract:
Return only the facts stated in this text as JSON. Do not interpret,
score or judge anything. Missing fields are null.

Call 2 — judge:
Here is a JSON object of extracted facts. Score it against these criteria:
[criteria]. Return a score and a reason per criterion. Do not re-read the
original text.
```

**Why:** when the single mega-prompt gets it wrong, you can't tell whether extraction or judgement failed. Split it and the failure has an address.

---

## 7. Facilitator crib sheet

### Timing

| Block | Minutes | Call time at |
|---|---|---|
| Warm-ups (W1–W6) | 10 | Do W1, W3, W5 live if the room is slow |
| Lab 1 — Stitch | 15 | Two-minute warning at 12 |
| Lab 2 — AI Studio Build | 20 | Two-minute warning at 17 |
| Lab 3 — Deploy | 15 | Cut the phone test first if you're behind |

### The five failures you'll actually see

| Symptom | Cause | Fix in one line |
|---|---|---|
| Stitch output looks generic | Style block used "modern" and "clean" | Make them name a colour, a shape and one thing to avoid |
| Stitch gives twelve mediocre screens | Asked for the whole app at once | Three screens. Regenerate. |
| AI Studio rebuilds the UI from scratch | Handoff prompt didn't say "keep the layout exactly" | Re-send §4.1 verbatim |
| App is pretty but has no AI in it | They skipped §4.2 | Ask them to say out loud what the one model call does |
| JSON parse errors halfway through | Asked for JSON in prose instead of a schema | Turn on structured output, §4.3 |

### What "done" looks like

- **Lab 1:** three screens on the canvas, same theme, code export in the clipboard.
- **Lab 2:** the app runs in preview and one real Gemini call changes something visible on screen.
- **Lab 3:** a URL that opens on someone else's phone.

### Before you leave the room

- Tell everyone to **rotate their API key** — it's been on screen, in screenshots, and possibly in a shared repo.
- Tell them to **set a budget alert** before attaching billing to anything.
- Point them at the 7-day challenge: one small tool, one URL, one real user.

---

*Build with AI Bootcamp · Prompt Wars · promptwars.in/bootcamp*
