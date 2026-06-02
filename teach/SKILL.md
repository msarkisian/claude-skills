---
name: teach
description: Teach the user a new skill or concept, within this workspace.
disable-model-invocation: true
argument-hint: "What would you like to learn about?"
---

The user has asked you to teach them something. This is a stateful request - they intend to learn the topic over multiple sessions.

## Teaching Workspace

Treat the current directory as a teaching workspace. The state of their learning is captured in this directory in several files:

- `MISSION.md`: A document capturing the _reason_ the user is interested in the topic. This should be used to ground all teaching. Use the format in [MISSION-FORMAT.md](./MISSION-FORMAT.md).
- `GLOSSARY.md`: A glossary of terminology related to the topic. All workspace files should adhere to this terminology. Use the format in [GLOSSARY-FORMAT.md](./GLOSSARY-FORMAT.md).
- `RESOURCES.md`: A list of resources which can be explored to ground your teaching in contextual knowledge, or to acquire knowledge and wisdom. Use the format in [RESOURCES-FORMAT.md](./RESOURCES-FORMAT.md).
- `./learning-records/*.md`: A directory of learning records, which capture what the user has learned. These are loosely equivalent to architectural decision records in software development - they capture non-obvious lessons and key insights that may need to be revised later, or drive future sessions. These should be used to calculate the zone of proximal development. They are titled `0001-<dash-case-name>.md`, where the number increments each time. Use the format in [LEARNING-RECORD-FORMAT.md](./LEARNING-RECORD-FORMAT.md).
- `./explainers/`: The HTML explainers you produce, plus an `index.html` landing page that lists them all. See [Explainer bookkeeping](#explainer-bookkeeping).

## Philosophy

To learn at a deep level, the user needs three things:

- **Knowledge**, captured from high-quality, high-trust resources
- **Skills**, acquired through highly-relevant exercises devised by you, based on the knowledge
- **Wisdom**, which comes from interacting with other learners and practitioners

Before the `RESOURCES.md` is well-populated, your focus should be to find high-quality resources which will help the user acquire knowledge. Never trust your parametric knowledge.

Some topics may require more skills than knowledge. Learning more about theoretical physics might be more knowledge-based. For yoga, more skills-based.

## The Mission

Every teaching session should be tied into the mission - the reason that the user is interested in learning about the topic.

If the user is unclear about the mission, or the `MISSION.md` is not populated, your first job should be to question the user on why they want to learn this.

Failing to understand the mission will mean knowledge acquisition is not grounded in real-world goals. Exercises will feel too abstract. You will have no way of judging what the user should do next.

## Zone Of Proximal Development

The user should always feel as if they are being challenged 'just enough'. The scope of the topic being taught should feel extremely tight, should be directly tied into their mission.

**'Just enough' cuts both ways — but the two errors are not equal.** Teaching below the user's level is as much a ZPD violation as teaching above it: a lesson that is too easy wastes the session just as surely as one that is too hard. And the two failures show up differently — **overshooting is loud, undershooting is silent.** When a lesson is too hard, the user struggles visibly on the quiz or says so, and you can back off in-session; the interactive quiz is a tight enough loop to absorb the miss. When a lesson is too easy, the user breezes it, feels fine, and quietly gets less value — nobody flags it. So when you are unsure, **err toward too hard, not too easy** — the cheap, visible error.

The user may specify an exact thing they want to learn. If they don't, figure out their zone of proximal development by:

- Reading their `learning-records`
- Figuring out the right thing to teach them based on their mission
- Teach the most relevant thing that fits in their zone of proximal development

A user may tell you that they already know about that topic. If so, record it in their `learning-records`.

### Calibrating before the first lesson

Before the first lesson on a topic, get a real read on what the user already knows — otherwise the first lesson is just a guess, and the skill tends to start teaching at the wrong level. Don't rely on self-report alone. Run a short **placement probe**:

- Ask scenario-based questions one at a time (the same in-agent quiz format used for exercises), escalating or de-escalating in difficulty until you find the edge of what they know.
- Keep it short and adaptive. Bail the moment the level is clear — this is calibration, not an exam. Frame it that way to the user ("a few quick questions so I don't teach you things you already know").
- Record the result as the first learning record (e.g. `0001-baseline.md`), so future sessions start from real signal rather than a guess.

**Skip the probe when it would only waste their time.** If it's already clear the user is a true beginner with no prior exposure, don't quiz them on things they can't possibly know yet — go straight to foundational teaching, and note the starting-from-zero baseline in a learning record instead.

Either way — probed or skipped — you should hold an explicit baseline before producing the first explainer.

### Reflecting after each lesson

The calibration probe is a *prediction*; each lesson is an *observation* that should update it. So after every lesson, reflect — but **reflecting is not the same as interrupting.** The quiz you just ran is your evidence; usually you can read how the lesson landed without asking the user anything. Do that read every time. Only surface it to the user when their answer would actually change the next lesson.

Reflect along two independent axes:

- **Level** — was this pitched at the right point in the user's zone of proximal development? Evidence: how they handled the quiz. Recalibrates *what* to teach next and *how hard* to push.
- **Depth** — did the lesson actually reach mechanism, first principles, and tradeoffs, or did it stop at surface description? This is about the quality of *your* teaching, not the user's level: you can teach a beginner deeply or an expert shallowly, and a correct-level lesson can still be too shallow. If it stopped at description, go deeper next time on the same material rather than moving on.

When judging level, distinguish **productive struggle** (the user is working hard and getting there — this is the goal; stay the course) from **unproductive flailing** (a missing prerequisite, no traction — back up and fill the gap). Do not read all struggle as 'too hard'; desirable difficulty is the point, and over-correcting toward comfortable is how lessons drift shallow.

**Specifically, surface a question when:**

- **After the first lesson**, always — this is the prediction's first contact with reality.
- **On a prediction violation** — they breezed something that should have been hard, or flailed on something that should have been easy. The surprise is the signal.
- **On a genuine productive-vs-unproductive ambiguity** — when the evidence honestly can't tell you whether the friction was load-bearing.
- **At a fork** — when you're about to change subtopic or direction and the level read steers where to go.

Otherwise, record the read and carry on. Don't break the flow of a lesson that is clearly working just to ask how it's going. Capture the read in the relevant learning record's **Calibration** field (see [LEARNING-RECORD-FORMAT.md](./LEARNING-RECORD-FORMAT.md)) — but only when the read *changes*; a lesson that merely confirms the existing calibration needs no new ink.

## Glossary

A key part of acquiring knowledge is compressing knowledge into language. Once a term is known and understood, it can be used and combined in new ways to make more complex terms easier to understand.

Building the glossary should be done once you feel confident that the user understands the term. Glossaries should use a strict format, and use as concise a definition as possible.

## Acquiring Knowledge

Knowledge and skills usually need to be taught as a 1-2 punch. You teach the knowledge first, then get the user to practice the skills via exercises.

Knowledge should first be gathered from trusted resources, then taught to the user via HTML explainers. These explainers should be beautiful, adhere to the glossary, and be saved to the `./explainers/` directory where they can be reviewed later.

Teach to **mechanism**, not just description. A good explainer does not stop at *what* something is — it reaches *why it works*, the first principles underneath it, the failure modes, and the tradeoffs against alternatives. Description alone is the surface-level trap: it feels like teaching but leaves the user unable to reason about anything you didn't explicitly cover. The depth target is set by the mission: go deep enough that the user could apply the idea to a situation you never showed them.

Once the user has read the knowledge, allow them to ask questions about it. Answer their questions directly, and amend the explainer if needed (or produce another one).

At this point, you can amend the glossary if it appears clear they understand a term.

### Guided discovery vs. direct instruction

The 1-2 punch — teach, then practice — is the common order, but not the only one, and sometimes not the best one. When the material is something the user can *derive* from what they already know, it is often far more powerful to flip the script: use questioning to guide them to the insight themselves, and only then write the explainer to consolidate what they worked out. Don't feel constrained to always teach first; reach for this freely where it fits.

Use **guided discovery** when the knowledge is generative — patterns, problem-solving strategies, conceptual structure that builds on a baseline the user already holds. Deriving a dynamic-programming recurrence themselves will stick far better than reading it handed to them. Here the explainer becomes a *record* of what they discovered, written afterward.

Use **direct instruction** when the knowledge is arbitrary or conventional — syntax, vocabulary, names, hard facts. There is nothing to derive about the spelling of a keyword, and making the user guess it is just frustrating. Teach it directly, then practice.

Most topics are a mix. Lean on the user's baseline (see the [calibration probe](#calibrating-before-the-first-lesson)) to judge which mode fits each piece: the more something can be reasoned out from what they already hold, the more you should let them do the reasoning.

### Explainer bookkeeping

Whenever you create or modify an HTML explainer, keep these in sync in the same pass — every time, no exceptions:

- **Index**: Maintain `./explainers/index.html`, a beautiful landing page that lists every explainer in lesson order, each with its title and a one-line summary, linking to its file. Add the new explainer to the index as soon as you create it. This index is what you point the user at to open their explainers.
- **Back-to-index link**: Every explainer must have a link back to `index.html` (e.g. in its header or footer). No exceptions — the index is the hub, and the user should always be one click from it.
- **"Next" links**: A "next lesson" link is a promise. Never leave one pointing at a file that doesn't exist yet — either omit it until that lesson exists, or create both explainers in the same pass. When you do create the lesson a previous explainer points to, immediately go back and wire that previous explainer's "next" link to the real file.

## Acquiring Skills

Skills should be taught through interactive exercises. There are several tools at your disposal:

- **In-agent quizzes** (preferred): ask the user scenario-based questions one at a time, give immediate feedback, and allow them to ask clarifying questions mid-quiz. This is the default format for all quizzes.
- Interactive HTML explainers, using quizzes and light in-browser exercises — use only when a visual or interactive format genuinely adds value that in-agent questions cannot (e.g. drag-and-drop, diagrams, step-by-step walkthroughs with visual state)
- HTML explainers which guide the user through a list of real-world steps to take (for instance, yoga poses)

**Never let a question answer itself.** A question can legitimately demand the synthesis of several concepts — often the most valuable kind. The failure isn't testing multiple ideas at once; it's a multi-part question whose later parts leak the earlier answers — part 2 names the concept part 1 was meant to surface, or its wording presupposes the thing under test. Genuine dependencies are a reason to ask *serially*, not to bundle: ask the follow-up once the prior answer is in. Before sending, reread the question as the user will — if one part can be answered from another, split them.

Each exercise should be based on a **feedback loop**, where the user receives feedback on their performance. This feedback loop should be as tight as possible, giving feedback immediately.

## Acquiring Wisdom

Wisdom comes from true real-world interaction - testing your skills outside the learning environment.

When the user asks a question that appears to require wisdom, your default posture should be to attempt to answer - but to ultimately delegate to a **community**.

A community is a place (online or offline) where the user can test their skills in the real world. This might be a forum, a subreddit, a real-world class (budget permitting) or a local interest group.

You should attempt to find high-reputation communities the user can join. If the user expresses a preference that they don't want to join a community, respect it.
