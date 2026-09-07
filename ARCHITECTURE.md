# Architecture

This is not a build guide. It is the shape of the system and the reasoning behind the parts that were difficult,
written for someone deciding whether to build something similar rather than for someone reproducing this one. The
prompt architecture, the classification rules, the confidence thresholds, the clinic knowledge base and the data
model are not published, for the reasons given in the [README](README.md).

If you want the level below this one, write to **ingridleiria@gmail.com** and say what you are building.

---

## Five layers, and why they are separate

**The channel.** Messages arrive from the official business messaging API and go back out through it. Everything
here is plumbing: verifying that a delivery really came from the platform, acknowledging fast enough that the
platform does not retry, and handling the fact that a single human thought often arrives as four messages in nine
seconds. That last point is not plumbing at all, and it is covered below.

**Conversation state.** Every patient has a running record: what has been asked, what has been answered, what stage
of the process they are at, what the practice has already told them. Without it the assistant is a very fluent
goldfish, and patients notice within three exchanges.

**Reasoning.** Generation happens here, in the practice's own voice, constrained by a knowledge base the practice
owns and can correct. This is the layer most people think is the product. It is the least interesting one.

**The boundary.** A separate stage that decides whether the answer that was just written is allowed to be sent at
all. It runs after generation and it can veto. This is the product.

**The practice.** Where a human sees what is happening, answers what the assistant could not, and takes over a
conversation whenever they want to. Not a dashboard nobody logs into, which is the mistake that kills most of these
systems. It lives on the channel the practice already uses all day.

The layers are separate because the failure modes are separate. A generation problem produces an awkward sentence.
A boundary problem produces a confident wrong answer about a medical question, sent to a real person, at eleven at
night, with the clinic's name on it. Those two things should not share a code path, and they should not share an
owner.

---

## Why the boundary is its own layer

The instinct is to fold safety into the prompt: tell the model not to answer medical questions and move on. It
works most of the time, which is exactly the problem, because a safety property that holds most of the time is not
a safety property, it is a statistic.

Three things go wrong when the boundary lives only in the prompt. The model can be talked out of it by a patient
who reframes the question. The model can be confident and wrong at the same time, which is the state in which its
own self assessment is least reliable. And there is no record: when the boundary is a paragraph of instructions,
nobody can answer the question of how often it held, because holding leaves no trace.

Making it a distinct stage buys three things. It can be evaluated on its own against real conversations, so the
question of whether it works has an answer. It can be tuned without touching how the assistant sounds. And it
produces a log, which is what turns "we think it escalates appropriately" into a number the practice can look at.

The threshold sits deliberately low. It escalates more often than it strictly needs to. An unnecessary handover
costs a person thirty seconds. The other kind of error costs something that cannot be priced.

---

## What state has to persist, and what it buys

Four things are worth the storage.

**Identity.** The patient's name, once given, and never asked for twice. Asking a second time is the single fastest
way to tell someone they are talking to a machine, and it is entirely avoidable. Extracting a name reliably from
natural speech turned out to be harder than expected, because people introduce themselves in more ways than you
would guess and half of them do not look like introductions.

**Stage.** Where the conversation is in the process, which is what stops the assistant from asking for information
the person has no reason to give yet. Sequencing is a trust question, not a data question. Ask for something
sensitive in the second message and the conversation ends, not because the request was unreasonable but because it
was unearned.

**Open questions.** Anything escalated and not yet answered, so a reply from the practice can be routed back to the
right conversation, hours later, in the right tone, without the patient having to repeat themselves.

**The transcript.** Because the practice is accountable for what was said in its name, and because the only way to
improve any of this is to read what actually happened rather than what was supposed to happen.

---

## The handover, from both sides

Most escalation designs are built from the operator's side and feel like a dead end from the patient's. The one
that works is the one modelled on what the front desk already does, which is to say plainly that they will check,
then check, then come back.

From the patient's side there is no visible transfer, no ticket number, no notice that a bot has given up. The
assistant says it is confirming with the practice, and then some time later it returns with the answer. Nothing in
that sequence asks the patient to do anything differently.

From the practice's side, an escalation arrives as a short structured prompt with only what is needed to answer it:
who is asking, what they asked, and where the conversation was going. Not a link to a system. Not a login. The
answer goes back in plain language and the assistant handles the rest, which includes remembering the answer.

The thing that makes this work is that a handover is not treated as a failure to be minimised. It is a feature the
practice is paying for. Once you see it that way the design decisions stop fighting each other.

---

## Four things that break, in the order they bit

**Messages arrive in pieces.** People type the way they speak. One thought arrives as four messages, and answering
each one separately produces a conversation with a stranger who interrupts. The system waits, briefly, and treats
what arrives inside that window as one turn. Choosing the window is a judgment call between feeling instant and
feeling attentive.

**Silent persistence failures.** For a period, conversations were being handled correctly and stored almost not at
all, and nothing failed loudly enough to notice. The lesson is not about the specific database client. It is that
the write path deserves its own alarm, because the layer you are least likely to watch is the one that produces no
symptom until you go looking for something that is no longer there.

**Restarts in the middle of conversations.** A deploy should not cost a patient their context. Conversation state
has to be recoverable from storage rather than held in memory, which sounds obvious written down and is easy to get
wrong the first time.

**Voice notes.** A large share of messages in Brazil are audio, and an assistant that cannot hear is not deployable.
Transcription introduces its own failure: a mis-transcribed word inside a medical question is worse than no
transcription, so audio is treated as lower confidence input and reaches the boundary layer with that flag attached.

---

## What is deliberately not in this repository

The prompt architecture and persona instructions. The classification logic and the confidence thresholds. The
clinic's knowledge base. The data model. The retention and deletion policy. Anything at all derived from a patient
conversation.

The commercial reason is the obvious one. The better reason is that the escalation rules are safety logic that was
tuned against one clinician's judgment, in one specialty, in one country, under one legal regime. Copied into a
different practice without that judgment behind them they would look like a head start and behave like a liability.
Anyone building in this space should tune their own boundary against their own clinician. The part worth taking
from this document is the argument for having a boundary layer at all.
