# How this was built

*An economist and a surgeon built a production assistant without hiring a developer. This is what that actually
took, including the parts that make it sound less magical than the phrase suggests.*

---

## The search that came first

Before any of this was software, it was a physician trying to buy a solution and failing.

He was losing patients he never met. Not in surgery, before it, in the gap between someone sending a message at ten
at night and someone walking into a consultation. That gap is filled with slow replies, inconsistent information
and messages that quietly go unanswered, and it costs a small practice more than any line item you could point at.

So he went looking. Pre built chatbots for messaging apps. CRM tools with an AI layer bolted on. General assistants
configured for healthcare. The local healthtech platforms. He tested all of it.

Everything had the same failure, and it was not a language quality problem. Patients would engage for one or two
messages, sense the script, and stop. Or worse, keep going while clearly feeling they were filling in a form rather
than talking to the practice.

The word he kept using was desumanizado. Dehumanised.

That word is the origin of everything that follows, because it names a problem that cannot be fixed by a better
model. It has to be fixed by someone willing to read hundreds of conversations and remove one machine tell at a
time.

---

## What was actually agreed, in a conversation with no technology in it

The first design session was not about architecture. It was about what the right experience should feel like for
someone messaging a surgical practice for the first time.

What came out of it was a short list that never changed:

- She should feel heard before she is asked for anything
- She should get real information, not a redirect to book a consultation
- She should be guided rather than interrogated
- If the assistant does not know something, it should say so and then go and find out
- She should never be uncertain about whether she is talking to a person

Read that list again and notice that four of the five are constraints on behaviour, not features. That ratio held
for the entire project.

---

## Who contributed what

This is a collaboration, not a vendor relationship, and the division is worth stating precisely.

**I contributed** the technical build, done through AI tools under close supervision; the product decisions about
what to build and in what order; the testing, which mostly means using the thing myself and noticing what felt
wrong; the iteration that turns "this response is off" into an instruction specific enough to change behaviour;
and the system design, including the escalation architecture that is the reason the project is defensible at all.
This is unpaid work, for reasons set out in [ABOUT.md](ABOUT.md).

**The practice contributed** the clinical knowledge, which is not a detail: what patients really ask, what the
honest answers are, where the line sits between orientation and medical advice. The process design. The feedback,
in the form of "we do not talk to patients like that" and "someone from here would phrase it this way." And the
history of real patient interaction that the assistant's instincts were built from.

Neither side could have done this alone, and I do not think that is a modest thing to say. I could not have known
the clinical nuance. They could not have built the system. The collaboration is the product.

---

## What real patients broke, immediately

Going live is where planning stops being useful. Four problems surfaced in the first weeks, none of which had come
up in testing.

**It asked for a sensitive identifier far too early.** Someone would say they were interested, and within two
messages the assistant wanted formal identification. They stopped replying. They had not decided whether they
trusted the practice yet, and the request was not unreasonable so much as unearned. The fix was to restructure the
whole sequence around when trust exists rather than around when the data is convenient to collect.

**It was too formal.** Full formal constructions, the greeting a bank uses. In that part of Brazil nobody speaks
like that, and the effect was to make a surgical practice sound like a call centre. The fix was explicit regional
tone guidance, which is a polite way of saying I wrote down dozens of specific phrasings and banned them by name.

**It forgot things it had been told.** A patient gave her name in the third message and was asked for it again six
messages later. Nothing destroys the illusion faster, and fixing it properly meant handling every way a person
might introduce themselves, most of which do not look like introductions.

**It answered a question it should not have answered.** Someone asked about an interaction with a medication and
got a confident, wrong reply. This is the failure that produced the escalation system, and it is the reason the
whole architecture is organised around a boundary rather than around answers.

---

## Humanisation is a practice, not a feature

There was never a version where humanisation was added. There was a weekly habit that never stopped.

Every week I read real conversations and find the moments where it sounded like software. Then I translate the
observation into something specific enough to act on. The instructions look like this:

> It opened with an acknowledgement token before answering. Real people do not do that. It reads as a support
> script. Remove that pattern completely.

> It listed three questions in a numbered format. That is a form. Ask one thing at a time, the way a conversation
> works.

> It closed with a formal sign off borrowed from email. Nobody writes that in a messaging app. Remove it.

> It identified the procedure correctly and then described it in clinical vocabulary. The patient used her own
> words for it. Use hers.

> This patient mentioned she had wanted this for five years and had finally decided. The reply acknowledged it in
> one clause and moved on to collecting information. It should have stopped there for a moment. That was the whole
> message.

Months of those accumulate into a personality. There is no shortcut, and there is no model release that does this
for you, because the raw material is your own clinic's conversations and nobody else has them.

---

## The escalation, which came from watching a person work

The most important decision in the system is also the least clever: let it ask for help.

Early versions tried to answer everything, which produced confident wrong answers, which is the worst available
outcome in a clinical setting. The fix came from watching how the practice coordinator actually works. When someone
asks her something she is not sure about, she does not improvise. She goes and asks the surgeon, then comes back.

The assistant needed the same move. The mechanism is described at the level worth publishing in
[ARCHITECTURE.md](ARCHITECTURE.md), and the important part is not mechanical anyway. It is that from the patient's
side there is no visible handover, no ticket, no notice that the software has given up. The assistant says it is
confirming, and then it comes back with the answer. From the practice's side, an escalation is a short structured
question that takes half a minute to answer, on the channel they already use.

Once you stop treating handovers as failures to be minimised and start treating them as the thing the practice is
paying for, most of the design tension disappears.

---

## What the practice got that nobody planned

As conversations accumulated, the coordinator needed visibility. The obvious answer was a dashboard, and the
obvious answer was wrong, because it would have meant asking someone who is busy all day to learn a new tool and
log into it.

So the assistant learned to talk to the practice too. Staff can ask it what happened today, who is waiting on a
reply, and what a particular conversation was about, in the same app, in plain language, and get an answer from
live data. Access is restricted to the practice, and the details of how are not published.

That feature was never on a roadmap. It came from a practical need, which is where most of the good ones came from.

---

## What vibe coding actually requires

The phrase makes it sound like you describe a thing and receive it. The description part is accurate. The
receiving part is not.

The hard part is not the code. It is the specification, and the taste to reject what comes back.

To get an AI to build something well you have to know what well means with enough precision to recognise its
absence. You have to notice when your intent was misread, which is harder when you cannot read the code, and it
means holding a clear model of the system in your head by other means. My own description of the role is ruthless
product manager. The tools write everything. I approve nothing that is not right.

Three examples of what an instruction looks like in practice:

*"A hundred and twenty one people have talked to this and six of them are in the database. Something is silently
failing on the write path."*

*"It loses the patient's name when a record already exists for them. Find the condition that skips extraction."*

*"It needs to know when it is talking to the practice rather than to a patient, and behave completely differently."*

Each of those starts as an observation, not a technical specification. Turning them into code was the tool's job.
Noticing was mine, and noticing is the part that does not automate.

---

## The numbers

| | |
|---|---|
| Concept | Late 2024 |
| First live patient | Early 2025 |
| Real patient conversations handled | More than a hundred |
| Running cost during development | Around forty five dollars a month in tooling |
| Approximate size | Several thousand lines |
| Lines written by hand | None |
| Developers employed | None |
| Weeks of reading conversations | Ongoing |

---

## Why any of this matters

This is not a demo, a prototype or a pilot. It is a system in production that has handled real patient
conversations, in Portuguese, about real procedures, with real consequences attached, in a regulated setting.

It was built by someone whose training is in economics, using tools available to anyone reading this.

That is the shift worth paying attention to. Expertise is becoming executable. The surgeon who knows what patients
need, the coordinator who knows how the process actually runs, the economist who knows what the data is for: they
can now build the thing that encodes what they know, without it being translated by someone who does not know it.

The caution that belongs next to that sentence is the entire reason the boundary layer exists. Lowering the cost of
building does not lower the cost of being wrong, and in a clinical setting the second number is the one that
matters.

If you want to talk about any of it, write to **ingrid@leiriaconsulting.com**.
