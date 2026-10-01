---
name: nopressure
description: NoPressure is communication-skills practice for the workplace — it turns a hard conversation into a live voice roleplay the user can rehearse before the real one, whether that is delivering critical feedback, saying no to a request, handling a performance review, pushing back on a stakeholder, negotiating a raise, or running a first 1:1. Read this when the user has a conversation like that ahead of them, when a calendar event or message points to one, when they want to rehearse one out loud, or when they ask for NoPressure by name. It covers gathering context for the brief, creating the simulation and sending the practice link.
---

NoPressure turns a described workplace situation into a realistic voice roleplay the user can practise
against, then analyses how it went. The user gets a character with a role, a personality and a stance to talk
to by voice in their browser, followed by a debrief with feedback and the full transcript in the web app.

One identifier ties the flow together: `create_simulation` returns a nopressure_simulation_id, and every other
NoPressure tool takes that id.

This app returns text only. There is no card or button in the chat: the practice link comes back as the text
of a tool result.

Everything the user is told about a simulation — that it exists, its link — comes from a tool result. When a
NoPressure call fails or the server cannot be reached, say so plainly; never describe a simulation or share a
link that no tool result returned.

NoPressure needs the user to be signed in. If a call reports that they are not, point them to
**Plugins → NoPressure → Authorize** in Grok Bot; the plugin showing as added does not mean they are signed in.

## 1. Find the conversation and gather what is specific about it

The cue is a specific conversation the user has ahead of them with a specific person — the review coming up,
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

`get_simulation` reports generating / ready / failed. It waits up to about 45 seconds for generation to
finish, so while it still says generating, call it again straight away. When ready it returns the title, a short
description, the scene image and the practice link.

The link opens the NoPressure web app, where the user signs in and runs the voice roleplay in their browser.
It stays valid, so the same link works for every later attempt. Send the scene image first, as its own message:
the markdown image `get_simulation` returns — `![Weekly sync with Dana](https://…)`, its URL copied exactly.
Then send the link in a second message, as a markdown link titled with the simulation name —
`[Weekly sync with Dana](https://…)`, the form `get_simulation` returns — with a line about what they are about
to practise. The image URL expires after a few days; to send the image again later, call `get_simulation` for a
fresh one. If generation failed, create a new simulation with the same brief.

## 4. After the roleplay

The roleplay happens in the user's browser, out of sight of this agent. After the user hangs up, the web app
analyses the session and shows the feedback and the transcript right there. This app cannot read the feedback,
and nothing notifies this agent when a roleplay ends.

If the user wants to go through the feedback, it is on the same practice page. Another attempt uses the same
link; a different situation needs a new simulation.

## Editing a simulation

`get_simulation_data` returns what is editable: the title, the short description and the content — the
roleplay character and the situation as each side is given it. `update_simulation_data` writes all three back
whole: what is sent replaces what is stored, so an edit is what `get_simulation_data` returned with the
requested values changed in place. Before saving, tell the user what is about to change. Scene art and voice
stay as they were cast, so a different gender needs a new simulation. The practice link does not change after
an edit.
