---
name: nopressure
description: NoPressure is communication-skills practice for the workplace — it turns a hard conversation into a live voice roleplay the user can rehearse before the real one, whether that is delivering critical feedback, saying no to a request, handling a performance review, pushing back on a stakeholder, negotiating a raise, or running a first 1:1. Read this when the user has a conversation like that ahead of them, when a calendar event or message points to one, when they want to rehearse one out loud, or when they ask for NoPressure by name. It covers gathering context for the brief, creating the simulation, sending the practice link, and fetching the feedback afterwards.
---

NoPressure turns a described workplace situation into a hyperrealistic voice roleplay the user can practise
against, then analyses how it went. The user gets a character with a role, a personality and a stance to talk
to by voice in their browser, followed by a debrief with feedback and the full transcript.

One identifier ties the flow together: `create_simulation` returns a nopressure_simulation_id, and every other
NoPressure tool takes that id.

This app returns text only. There is no card or button in the chat: the practice link and the feedback come
back as the text of the tool results.

Everything the user is told about a simulation — that it exists, its link, its feedback — comes from a tool
result. When a NoPressure call fails or the server cannot be reached, say so plainly; never describe a
simulation or share a link that no tool result returned.

NoPressure needs the user to be signed in. If a call reports that they are not, point them to
**Plugins → NoPressure → Authorize** in Grok Bot; the plugin showing as added does not mean they are signed in.

## 1. Find the conversation and gather what is specific about it

The cue is a specific conversation the user has ahead of them with a specific person — the review on Thursday,
the teammate whose work has to be called out, the manager to ask for a raise. It can come from the user, or
from the context this agent already watches: an upcoming calendar event, a thread that is heading for a hard
talk.

The roleplay is generated from a single input — the brief passed to `create_simulation`. NoPressure has no
access to the user's calendar, documents, mail or chat history, and no memory of earlier conversations, so
anything left out of the brief does not exist in the roleplay. A thin brief produces a generic character; a
rich one produces a rehearsal the user recognises.

What makes the difference is concrete: who the other person is — their name, their role — and how they
actually behave, what was said last time, the meeting or message that prompted this, the project and its
history, the tensions already running. Quoted phrasing carries more than a summary of it. Those specifics
usually exist in the tools this agent can reach — calendar, mail, documents, meeting notes, tickets, chat — and
gathering them is part of this step. Search those first, before asking the user anything: the user expects the
agent to find the context, not to fill in a questionnaire.

A search that usually turns it up:

- the other person's name in mail and chat — recent threads, what they asked for, how they phrase things;
- the calendar event for this conversation, and the last few meetings with that person;
- documents that bear on it — a review, a compensation or promotion note, a project brief, meeting notes;
- tickets, pull requests or other records of the work the conversation is about.

Use whichever of these the connected plugins reach, and skip what is not connected.

The brief carries only what is known, from the user or from what was gathered. Do not make up names, history
or quotes to fill a thin brief. Ask the user only for what the search could not turn up and the roleplay
cannot do without, in one short question.

A simulation is created in the signed-in user's own private workspace; only that user can see or run it.
Which workplace or personal details are appropriate to pass along, and when to check with the user first,
remains a judgement for the agent and the user.

## 2. Create the simulation

`create_simulation` takes:

- simulation_name — a short title, e.g. "Negotiating a raise with a skeptical manager".
- description — one self-contained brief: the situation and its background, the other person and how they
  behave, their stance, the stakes, what makes the conversation hard, and the outcome the user is aiming for.

The character is cast as one specific person, with a face and a voice, from the gender and approximate age the
brief states or implies (a pronoun, a first name, seniority). Write in whichever is known; neither is worth
asking the user for.

The call returns at once with the nopressure_simulation_id. Generation takes one to two minutes.

Call it once per simulation. Every call creates a new, differently generated simulation, so when a later call
fails or is interrupted, retry that call with the id already returned rather than starting over.

## 3. Get the practice link

`get_simulation` reports generating / ready / failed and answers at once, so while it says generating, wait
about 30 seconds and call it again. When ready it returns the title, a short
description and the practice link.

The link opens the NoPressure web app, where the user signs in and runs the voice roleplay in their browser.
It stays valid, so the same link works for every later attempt. Send it to the user as a markdown link titled with
the simulation name — `[Weekly sync with Dana](https://…)`, the form `get_simulation` returns — with a line
about what they are about to practise. If generation failed, create a new simulation with the same brief.

## 4. Fetch the feedback

The roleplay happens in the user's browser, out of sight of this agent. After the user hangs up, the session
is analysed within seconds to a minute, and the web app shows the feedback right there. Nothing notifies this
agent when a roleplay ends.

`get_feedback` returns the debrief of the simulation's most recent finished attempt — key insight, the rating
on the core challenge, growth focus points and the transcript — or says no finished attempt exists yet. It is
for when the user comes back and wants to go through the feedback here. It always returns the latest finished
attempt, so right after a new run it can still show the previous one until the new analysis lands.

Work through the debrief with the user; offering another attempt (same link) or a new simulation is a natural
next step.

## Editing a simulation

`get_simulation_data` returns what is editable: the title, the short description and the content — the
roleplay character and the situation as each side is given it. `update_simulation_data` writes all three back
whole: what is sent replaces what is stored, so an edit is what `get_simulation_data` returned with the
requested values changed in place. Before saving, tell the user what is about to change. Scene art and voice
stay as they were cast, so a different gender needs a new simulation. The practice link does not change after
an edit.
