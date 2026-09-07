# VIZZ

**A WhatsApp assistant in production at a private surgical practice in southern Brazil.**

---

## The problem it was built for

A small specialist clinic receives more messages than a front desk can answer well. Most of them are the same twenty
questions asked in a hundred different ways, at hours when nobody is at the desk. The cost of answering slowly is
not only a lost booking. It is a patient who stays anxious overnight, or who asks somewhere less careful.

Off the shelf chatbots fail here for a reason that has nothing to do with language quality. They are built to always
have an answer. In a clinical setting, always having an answer is the failure mode.

## What VIZZ is

An assistant that handles the ordinary parts of patient communication in Portuguese, on the channel patients already
use, and that is designed around the moment it should stop.

- **It answers what is safely answerable.** Practical questions, orientation, what to expect, what to bring, how the
  process works. Written in the clinic's own voice rather than a generic support register.
- **It escalates rather than guesses.** When a message needs clinical judgment, or when the model's confidence in
  its own answer is not high enough, the conversation is handed to the practice. No hedged medical answer is better
  than a confident wrong one, and the assistant is built to prefer silence over improvisation.
- **It keeps the record.** Conversations are stored so the practice can see what was said, review anything, and
  intervene at any point.
- **It respects the rules it operates under.** Brazilian data protection law shapes what is collected, how long it
  is held, and what leaves the system.

## The design decision that matters

Everything else in this project is ordinary engineering. The part worth writing down is the boundary.

A clinical assistant is not a question answering system with a medical vocabulary. It is a triage system whose main
job is to recognise the cases it must not handle. That means the interesting work is not in the answers, it is in
the classification that happens before an answer is written, in the confidence threshold that decides when to hand
over, and in making the handover feel like care rather than a dead end for the person on the other side.

An assistant that answers ninety-five percent of messages well and improvises on the other five percent is worse
than one that answers eighty percent and says clearly that a human will reply to the rest.

## Where this can go

Designed and not deployed: multilingual handling for international patients, post-procedure follow up on a schedule
set by the practice rather than by the patient's anxiety, structured intake before a first consultation, and a
practice-side view of what patients actually ask, which is a research question of its own. Each of these raises the
clinical stakes, so each waits until the escalation behaviour has been observed for longer.

## How it is built, at the level worth publishing

A messaging integration on the official business API, a language model provider for generation, a relational
database for conversation state and audit, and an administrative interface for the practice. Deployed as a small
always-on service.

The prompt architecture, the classification and escalation logic, the confidence thresholds, the clinic-specific
knowledge base and the data handling policy are not published here. That is deliberate, and not only for commercial
reasons: the escalation rules are safety logic tuned against real clinical judgment, and a copy of them, applied to
a different practice without that judgment behind it, would be a liability rather than a starting point.

Nothing in this repository contains patient data.

## Status

In production. Built with the practice, and running against real patient conversations, which is why the safety
behaviour is treated as the product rather than as a feature of it.

## If you want to work with this

I am open to conversations about:

- **Clinical deployment.** If you run a practice and want something like this, the useful conversation is about
  where your own boundary sits, not about the software.
- **Research.** Patient communication patterns and AI triage boundaries in small practices is an under-studied area
  and I would like to work on it properly.
- **The safety design specifically**, if you work on escalation and confidence calibration for assistants that
  operate in regulated settings.

Write to **ingridleiria@gmail.com**.

## About

Built by [Ingrid Leiria](https://ingridleiria.github.io) with a physician collaborator, as a working answer to a
real operational problem rather than a demonstration. I write about this kind of work at
[BreakTalk](https://breaktalk.substack.com).

## Licence

Copyright 2026 Ingrid Leiria. All rights reserved. See [LICENSE](LICENSE).
