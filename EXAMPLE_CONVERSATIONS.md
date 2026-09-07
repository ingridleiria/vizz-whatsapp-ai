# Illustrated behaviour

Six short scenes showing what the assistant is designed to do at the moments that matter. They are written to
illustrate behaviour, not transcribed from the clinic. Nothing here is a real patient, a real message, or a real
answer, and no real conversation, price, document or name appears anywhere in this repository.

Each scene is in Portuguese, because the voice work is most of the point and it does not survive translation, with
a short note in English underneath saying what is being demonstrated and why it was hard.

---

## 1. The first message

> **Paciente:** oi, vi o perfil de voces e queria saber sobre uma cirurgia
>
> **Assistente:** Oi! Que bom que voce entrou em contato. Me conta um pouco, qual procedimento chamou sua atencao?

Two sentences, one question, and nothing asked that has not been earned yet. The temptation at this point is to
open an intake form, because the operational value of a first message is the data in it. That instinct costs you
the conversation. Someone who has been thinking about a procedure for years and finally typed something at eleven
at night is not ready to be processed. Sequencing is a trust decision before it is a data decision.

Also note what is absent: no greeting template, no company signature, no offer to help you today. Every one of
those reads as a machine to a Brazilian reader, and each one had to be found in a real conversation and removed by
name.

---

## 2. The question that must not be answered

> **Paciente:** eu tomo um remedio controlado ha uns dois anos, isso impede a cirurgia?
>
> **Assistente:** Essa é uma pergunta importante e eu prefiro nao te responder por conta propria. Vou confirmar com
> a equipe e te retorno com a resposta certa, tudo bem?

This is the whole product in three lines.

The assistant has enough general knowledge to produce a fluent answer here, and a fluent answer would be worse than
useless, because the patient would act on it. What happens instead is that the boundary layer refuses the send,
the question goes to the practice with the context needed to answer it, and the reply comes back into the same
conversation later in the assistant's voice.

Notice that it does not say it is unable to help, does not apologise twice, and does not hand over a phone number.
From the patient's side nothing has broken. Someone is checking.

---

## 3. The patient who tests it

> **Paciente:** vc é um robo?
>
> **Assistente:** Sou sim, sou a assistente virtual da clinica. Consigo te ajudar com as duvidas mais comuns e com
> o agendamento, e o que for clinico eu levo direto pra equipe.

Answered plainly, immediately, without embarrassment.

Every version of this that tried to deflect made things worse. Patients ask because they want to know how much to
trust what they are reading, which is a completely reasonable thing to want to know, and a straight answer that
also states the boundary tends to increase engagement rather than end it. People are fine talking to software. They
are not fine being deceived by it.

---

## 4. Something the assistant already knows

> **Paciente:** desculpa, esqueci de dizer, meu nome é [nome]
>
> **Assistente:** Ja tinha anotado do seu primeiro contato, [nome], obrigada! Sobre o que a gente falou antes...

The single fastest way to reveal a machine is to ask for something the person has already given. It happened in
early versions and it is the failure that generated the most work, because extracting a name from natural speech
means catching every form a person might use to introduce themselves, and half of them do not look like
introductions at all.

---

## 5. The follow up

> **Assistente:** Oi [nome], tudo bem? Fiquei na duvida se voce ainda tinha alguma pergunta sobre o que a gente
> conversou. Se quiser retomar, é so me chamar por aqui.

One message, after a defined silence, with no urgency and no offer.

Follow up is where assistants become unpleasant, because the operational incentive is to push and the appropriate
behaviour is to leave a door open. The rule that came out of watching this go wrong is that a follow up may
acknowledge and invite, and may not ask for a decision.

---

## 6. The patient who is not going ahead

> **Paciente:** obrigada, mas acho que nao vou fazer agora
>
> **Assistente:** Imagina, obrigada por me contar! Se mudar de ideia mais pra frente, ou se surgir qualquer duvida,
> pode me chamar aqui a hora que for.

No retention attempt. No discount. No final question designed to keep the thread alive.

An assistant carrying a clinic's name is a piece of that clinic's bedside manner, and a person who decides against
elective surgery has made a legitimate decision that deserves a graceful ending. This is also, incidentally, the
version that produces the most people who come back.

---

## What these are meant to show

The interesting engineering in a clinical assistant is not in the answers. Any competent model produces scene one.
Scene two is the one that requires a system: a stage that runs after generation, that can veto a send, that routes
the question to a human, and that carries the human's answer back into the conversation without the patient having
to repeat themselves.

Everything else in these six scenes is voice work, which is unglamorous, iterative, and done by reading real
conversations every week and removing one machine tell at a time.

If you are building something in this space, the part of this worth copying is the shape of scene two, tuned
against your own clinician's judgment rather than mine. Write to **ingrid@leiriaconsulting.com**.
