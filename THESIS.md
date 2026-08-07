# Systems of Agency

### Cloudflare OS and the Governance of Artificial Actors — a thesis in four connections, and an argument for the missing third passport

*Draft — August 2026*

---

## Abstract

The obvious reading of Cloudflare OS — "an enterprise ChatGPT where agents can make little apps" — misses almost everything interesting about it. Read closely, the system is an attack on three boundaries that have defined enterprise computing for thirty years: user versus developer, application versus document, and human versus software process. But the deeper finding, defended here with evidence from the codebase itself, is architectural: Cloudflare OS is the first substantial industrial implementation of a **System of Agency** — a governed layer that sits between human intention and the Systems of Record, managing not what *is* true but what *can become* true.

The thesis proceeds in four connections. First, the system's Gatekeeper layer constructs a technical boundary between an AI's *imagination* and real-world *consequence*, implemented so radically that the agent cannot distinguish acting on the world from acting on a simulation of it. Second, this boundary is not an accidental discovery but the deliberate application of a fifty-year-old minority tradition — object-capability security — whose designers always understood themselves to be modelling human institutional authority, and which has finally found the actor population it was waiting for. Third, the system implements the beginnings of a genuinely under-deployed governance primitive, here called **epistemic authority**: policy that follows what the agent has *seen*, so that an actor's accumulated knowledge state — not merely its identity or role — determines what it may subsequently cause and who may subsequently observe its work. Fourth, mapping the system against a three-part taxonomy of agent governance — the **authority passport** (what may I touch?), the **knowledge passport** (what has entered my reasoning?), and the **outcome passport** (what did I actually cause, and did the promised condition become true?) — reveals that Cloudflare has built the first strongly, the second conservatively, and the third not at all. Yet the third passport is *latent* in the architecture: the simulation machinery already produces, as exhaust, a machine-executable specification of each action's intended effect. Retaining that specification across the commit boundary and reconciling it against the realized world is the unbuilt organ — the back office of the System of Agency — and it is the foundation on which artificial actors become not merely trustable by institutions, but *underwritable by markets*.

---

## 1. Against the obvious reading

Cloudflare describes the project, in its own README, as "an operating system for AI productivity": an agent chat UI, sandboxed application development, and a security framework "such that non-technical users can safely 'go nuts' and nothing bad will happen."¹ The surface presentation invites a dismissive gloss — another chatbox with connectors, plus app generation.

That gloss fails on contact with the architecture. Three boundaries, each load-bearing in enterprise computing since the client–server era, are deliberately collapsed:

**User ↔ Developer.** A Gadget — the system's unit of software — is created as casually as a document, but carries UI, server code, durable state, real-time collaboration, and mediated access to enterprise systems. Each user gets a private, isolated instance, and the resident agent can modify its code on request. The README is explicit about the intended consequence: "the centralized model of software stops making sense" when any user can prompt an agent to add the features they need.¹

**Application ↔ Document.** The README's own analogy is an office suite in which "each file — or 'Gadget' — is potentially its own custom application, written by AI to serve exactly your needs."¹ Blueprints — shareable snapshots of a Gadget's code — complete the collapse: sharing a template now means sharing an application.

**Human ↔ Software process.** The approval architecture (Section 4) restructures the relationship between human judgment and machine execution — not by inserting a human *into* the loop, but by making the human the *commit phase* of a transaction the machine runs speculatively.

The consequence of the first two collapses can be stated in one line: **software is beginning to behave like language.** Today one writes a document because some representation does not yet exist; tomorrow one may generate software because some capability does not yet exist. The application ceases to be a long-lived institutional artifact and becomes an ephemeral expression of intention.

Two amendments keep this claim honest.

*The trust-cost inversion.* Casually-created software never failed for want of creation tools — spreadsheets made software behave like language forty years ago, and enterprises are consequently full of load-bearing spreadsheets nobody can audit. It failed because every new piece of software is a new attack surface and a new governance obligation. AI has collapsed the marginal cost of *writing* software while doing nothing to the marginal cost of *trusting* it. The distinctive engineering investment in this codebase is therefore not the generation pipeline but the trust machinery: per-instance isolation, default-deny networking, capability-based introduction of resources. The boundary collapse is only viable because the trust model was rebuilt first, and any successor system that copies the gadgets without the gatekeepers will rediscover why.

*The layer split.* Even inside Cloudflare OS, the Systems of Record remain external, centralized, and long-lived; Gadgets reach GitHub, Google, and Slack *through* Gatekeepers rather than replacing them. What becomes ephemeral and personal is the *interface and logic* layer; the *state and authority* layer stays institutional. That is a narrower claim than "the application dies," but it is still most of what SaaS vendors charge for.

---

## 2. The System of Agency

Arrange the architecture vertically and a new layer becomes visible:

```
        ERP · CRM · Core Banking · HR · GitHub · Email
                          │
                 SYSTEMS OF RECORD          — preserve what is true
                          │
                     Gatekeepers            — the controlled consequence boundary
                          │
            Agents · Gadgets · Workflows
                          │
                 SYSTEM OF AGENCY           — manages what can become true
                          │
                   Human intention
                          │
              Organizational consequence
```

A System of Record answers: *what is the case?* Its virtues are durability, consistency, and auditability of state. A System of Agency answers a different question: *what is permitted to become the case, by which actors, knowing what, under whose review?* Its subject matter is not state but **potential consequence** — the space of actions an artificial actor may imagine, the subset it may realize, and the conditions of that realization.

An agent workspace in Cloudflare OS contains persistent state, files, resource access, an execution environment, organizational context, and skills; it can research, build artifacts, create applications, and run workflows. Every one of its paths to the outside world runs through a Gatekeeper. The layer between intention and record is therefore not a pipe but a *jurisdiction* — and the remainder of this thesis is an anatomy of how that jurisdiction governs.

---

## 3. Connection one: the boundary between imagination and consequence

A Gatekeeper looks, at first glance, like a connector: a per-service Worker handling OAuth, wrapping the vendor API, logging calls. The project's own developer documentation dispels that reading. A Gatekeeper's API contract imposes a hard ontological split on every operation:²

- Every **observation** — any read of external data — must call `authorizeObservation()` before returning data to the caller.
- Every **action** — any externally visible side effect — must be submitted via `submitAction()` and "must not actually be performed until `applyAction()` has been called."

The documentation calls the capability API design "the most important and delicate part" of building a Gatekeeper, and mandates a hard stop for human review of the API before implementation proceeds.² This is the tell. The API is not merely an interface; it is **the vocabulary of consequences this artificial actor is capable of producing**. Nothing outside that vocabulary can be caused, because nothing outside it can be *expressed*.

Which yields the principle in one sentence: *an LLM may imagine an action; only an institutionally governed system may make that imagination real.*

The implementation is more radical than the principle. Traditional human-in-the-loop approval is a blocking syscall — the agent stops, the human approves, the agent continues — which is precisely why practitioners surrender to auto-approve modes. Cloudflare's alternative: when an action requires approval, the Gatekeeper **simulates** its outcome. Reads issued after a pending action must reflect that action's effects as if applied. The agent sends the email, reads back the thread, sees its email there, and proceeds — and all of it may be a speculative branch a human later aborts, at which point the Gatekeeper rebuilds its cache as if the imagined world had never existed. The developer docs go further: agent-facing API documentation is *forbidden* from mentioning the approval queue at all — "correct simulation keeps this invisible."²

So the design goal is that the agent *cannot tell* whether it is acting on the world or on a simulation of the world. Architecturally, **the agent is a transaction; the human is the commit phase.** The distinction between imagination and consequence is deliberately erased for the agent so that it can be preserved for the institution. The human's role shifts from synchronous supervisor to asynchronous reviewer of a described batch of consequences — a wager that reviewing forty proposed effects at leisure beats approving forty actions in real time.

### 3.1 Not an accident: the tradition

It is tempting to say Cloudflare *accidentally rediscovered* institutional authority. The history says otherwise. The observation/action split, the object-per-resource API design, the principle that *holding a reference is the authorization* — this is the object-capability security model, a lineage running from Dennis and Van Horn (1966) through Mark Miller's E language to Cap'n Proto. Cap'n Web, the RPC protocol every Gadget and Gatekeeper is required to speak, is Kenton Varda's work — the same lineage that produced Sandstorm, whose thesis a decade ago was per-user application instances with fine-grained capability grants. Cloudflare OS is recognizably that bet, retried.

More to the point: the capability community argued for decades, explicitly, that capabilities work because they mirror how human institutions actually delegate authority — power is acquired by being *introduced* to it by a party who legitimately held it, not by a central oracle consulting a list. Institutions were the design inspiration all along. What is new is not the idea but the actor population: AI agents finally supply actors numerous, fast, and untrusted enough to make a fifty-year-old minority tradition economically necessary. *The killer app for capability security turned out to be governing artificial actors.*

---

## 4. Connection two: epistemic authority

Security models have historically asked a short series of questions. Identity-based access control asks *who are you?* Role-based control asks *what role do you hold?* Capability security asks *what exactly have you been permitted to touch?* Cloudflare OS operationalizes a fourth:

> **What do you now know? And given what you know, what are you allowed to cause — and who is allowed to see your work?**

Call this **epistemic authority**: authorization as a function of accumulated knowledge state, not merely identity, role, or grant.

### 4.1 The mechanism

The observer-tracking design³ implements it concretely:

- Each Gatekeeper records what the Gadget has observed through it — at "data set" granularity where the vendor's ACL model permits (BigQuery datasets, Notion pages, Linear teams, Confluence spaces).
- When the Gadget is shared, every prospective collaborator must prove — *through their own connected account, verified inside the vendor's own trust domain* — that they could directly read everything the Gadget has historically read. The check (`addObserver`) re-runs on **every open**, so revocation at the source propagates.
- A maximally sensitive observation sets `prohibitAllSharing`, dropping the Gadget into lockdown: no sharing, no further actions, **no web fetches**.³ Observation history thus gates not only disclosure but outbound consequence.

The employee of a bank may ordinarily be permitted to send email. But an agent assisting that employee which has just ingested unreleased financials, HR records, customer PII, and a suspicious-transaction report is a different governance object. The question is no longer *may this employee send email?* but *may an actor possessing this particular state of knowledge transmit to this particular consequence surface?* Cloudflare OS is the first industrial system to make that question mechanically answerable.

### 4.2 The flow runs backwards

The v1 design contains a detail stronger than the headline. There is no per-observer hiding of individual observations; therefore, when the agent attempts to read data that a *current, still-authorized* collaborator could not see, the system does not filter the collaborator — it **blocks the observation**.³ The agent is denied the read.

Epistemic authority is therefore bidirectional. Policy follows what the agent has seen — and *what the agent may come to know is constrained by who is watching*. The audience's minimum clearance bounds the actor's perception, exactly as a classified briefing halts when someone uncleared enters the room. No prior access-control tradition in commercial software expresses this; rooms and meetings have always enforced it.

### 4.3 The lineage, again — and the honest limits

As with capabilities, the primitive is older than it looks. Labels attached to data, propagated through computation, checked at egress: this is decentralized information flow control (Myers & Liskov, 1997) and a decade of research operating systems — Asbestos, HiStar, Flume — none of which reached mainstream practice. Two reasons it failed then, and one reason it becomes mandatory now:

*Then:* tracking labels through computation was costly and porous, and monotonic label accretion ("label creep") drove every long-lived system toward maximally-restricted uselessness.

*Now:* with an LLM in the loop, tracking influence through the computation is not expensive but **impossible**. One cannot audit whether the unreleased financials influenced the dashboard's phrasing; the model launders everything it reads. Conservative boundary-level tainting — treating the whole Gadget as the unit of contamination — is not one design option among several. It is the only sound one.

Three limits must be carried honestly:

1. **This is not lineage through cognition.** The taint attaches to the *container* of cognition, not to actual influence. An agent that reads an HR file and then builds a tic-tac-toe game in the same Gadget has produced a restricted tic-tac-toe game. Over-approximation is the engineering response to an unauditable influence-mixer at the center of the system.
2. **The scheme is parasitic on the Systems of Record.** `addObserver` can only verify what the vendor exposes an oracle for. Where none exists (Home Assistant's all-or-nothing token; ZoomInfo's entitlements), the design table itself degrades to "no tracking" or "never share."³ The System of Agency does not define its own ontology of sensitivity; it *imports* one, vendor by vendor. Its governance is federated, not sovereign — and only as fine-grained as the stodgiest upstream ACL model.
3. **The accounting has an edge.** Tokens flowed through an LLM provider; the agent's conversational context is itself a channel the observer machinery does not label. The boundary is sound; the boundary's *location* excludes some real flows.

### 4.4 Two syntheses

**Disposability makes information flow control livable.** Label creep killed IFC in the era of long-lived applications, because restrictions accumulate monotonically and there is no cheap reset. But Section 1's collapse — software as ephemeral expression of intention — supplies exactly the reset: when the tainted unit is a disposable Gadget, one does not declassify the old container, one spins up a clean one and introduces only what it needs. The document/application collapse and epistemic authority are not two separate innovations; each is the condition of the other's viability.

**This is the mechanization of the insider list.** Banking already runs epistemic authority on humans: insider lists, ethical walls, trading windows that close the moment one attends the M&A meeting. MNPI regimes are precisely "given what you now know, what may you cause" — enforced by compliance officers, self-declaration, and sanctions, and therefore affordable only for a few hundred senior insiders. Cloudflare OS mechanizes the insider list: per actor, per Gadget, per observation, re-checked on every open — cheap enough to apply to *every* actor. Institutions always knew that knowledge changes authority; they could never afford to enforce it at scale. Artificial actors, whose knowledge state carries a machine-readable history that no human employee ever had, make the enforcement not only affordable but exact.

---

## 5. Connection three: the three passports

The governance of any actor — human or artificial — decomposes into three questions, in three different modalities:

| Passport | Question | Modality | Institutional analog |
|---|---|---|---|
| **Authority** | What am I allowed to touch and do? | Deontic (*may*) | Mandate; delegated powers |
| **Knowledge** | What information has entered my reasoning environment? | Epistemic (*knows*) | Insider list; need-to-know |
| **Outcome** | What did I actually cause — and did the promised condition become true? | Causal (*did*) | Audit; settlement; reconciliation |

Measured against this taxonomy, the state of Cloudflare OS is precise:

**Authority: strong.** The capability model of Sections 3–3.1 — introduction-based grants, object-per-resource APIs, fine-grained resource granularity, default-deny everything.

**Knowledge: real, conservative, v1.** The observer machinery of Section 4 — container-level tainting, per-open re-verification, forward exclusion — with the stated limits.

**Outcome: absent as a primitive.** The evidence is in the interface itself. `applyAction(action): Promise<void>` — the commit phase of the entire consequence pipeline **returns nothing**.⁴ No result, no post-state, no divergence report. The `ActionDescription` that constitutes the action's "promise" is designed, per its own JSDoc, for three uses — approval routing, display to the approver, audit log — and the promise itself is *prose*: "a complete description of the action... in Markdown-formatted natural language."⁴ There is a `revertAction()`, but it is user-triggered regret, not system-triggered reconciliation: best-effort compensation that may fail ("too many other changes may have been made"), offered as a UI affordance.⁴

The system therefore holds a **ledger** (every action submitted, described, approved, logged) and a **prediction** (the simulation), but no primitive that ever *closes the loop* — nothing that re-observes the world after apply and asks whether the promised condition became true.

The institutional mapping makes the gap vivid. Banking runs the three passports as front office (acts), middle office (limits, conflicts, walls — passports one and two), and back office (settlement, reconciliation, break resolution — passport three). By that map, **Cloudflare OS is a System of Agency with a front office and a strikingly sophisticated middle office, and no back office at all. Every trade executes; no trade ever settles.** In any institution of consequence — core banking above all — unreconciled action is not a missing feature. It is a finding.

---

## 6. Connection four: the latent passport, and Assured territory

### 6.1 The outcome passport is latent, not absent

Here the thesis turns from criticism to opportunity, because the third passport is already present in the architecture — as an unexploited byproduct of the first.

Consider what the simulation requirement forces every Gatekeeper to build. To simulate a pending action so faithfully that the agent cannot detect it has not happened, the Gatekeeper must maintain a machine-executable model of the action's expected effect on the world — an imagined post-state concrete enough to answer arbitrary reads against. **That is a formal specification of intent.** Cloudflare built it so agents would not block on approval; the codebase then reconciles it into the cache and forgets it at the moment of `applyAction()`.

An assurance layer does not need to invent the intent representation — the genuinely hard part. It needs to *retain* the simulated delta across the commit boundary, re-observe the System of Record, and diff the realized world against the imagined one. Divergence then becomes a first-class event: trigger `revertAction()`, escalate to the human, annotate the audit record with *promised vs. realized*. The imagination/consequence boundary of Section 3 already produces, as exhaust, exactly the artifact the outcome passport requires.

### 6.2 Why it is structurally different: gates versus loops

The first two passports are enforceable at the boundary *by construction*: withhold the capability, block the observation — synchronously, with certainty. The third cannot be a gate, because the external world is not transactional. Services fail partially; other actors race; effects are eventually consistent; and "the promised condition became true" is knowable only by *going back and looking* — itself an observation, subject to passport two, possibly stale on arrival.

Assurance is therefore necessarily a **loop**: a recurring reconciliation process with its own schedule, failure modes, and epistemics. And it wants separation of duties — the agent that acted is the wrong party to attest its own outcome. The natural attestation point is the Gatekeeper, which uniquely holds both the promise (its own simulation) and independent re-observation access. Institutions discovered this asymmetry centuries ago; it is why authorization lives in the front office while *assurance became a separate profession*. "Assurance" is, precisely, the auditor's term of art for independent verification that a representation corresponds to reality.

### 6.3 From dossiers to passports: the underwritable agent

One amendment to the metaphor, and it is the commercially decisive one. A passport is a portable credential the actor carries across borders. What Cloudflare OS holds today are **dossiers** — institution-held records, meaningful only inside the boundary that keeps them. They become passports only when histories turn into portable, verifiable attestations:

> *This agent, over N thousand applied actions across these Gatekeepers, diverged from its promised outcomes at rate X, with severity distribution Y, and compensated cleanly in Z% of divergences.*

Notice what that document is: a **loss history**. And an actor with a machine-attested loss history is an *underwritable* actor.

That is the endgame the outcome passport implies — not merely audit, but **surety**: bonding agents, pricing their risk, insuring principals against divergence between promise and outcome. Human institutions could never underwrite employees this way; human outcome histories are sparse, self-reported, and unportable. An agent's can be dense, attested at the Gatekeeper boundary by a party structurally separated from the actor, and cryptographically portable across Systems of Agency.

The progression, stated once:

- Passports one and two are how an **institution** trusts an agent.
- Passport three is how a **market** does.

This is Assured territory. The assured, after all, is the party the policy protects.

---

## 7. Objections and open problems

A thesis this convenient should list its own weaknesses.

**The rubber stamp relocates.** Batch-asynchronous approval defeats the auto-approve pathology by moving review to a convenient time — and thereby reintroduces rubber-stamping at the new location. Maker-checker regimes in banking, which the approval queue generalizes, have well-documented rubber-stamping pathologies. The wager that reviewing described consequences at leisure beats synchronous approval is plausible and unproven at organizational scale.

**The constitution is hand-written.** The "vocabulary of consequences" is only as honest as the description strings Gatekeeper authors write, and only as complete as their simulations. The project's own docs flag API design as the most delicate step and mandate human review² — an acknowledgment that the constitutional layer is drafted by fallible developers under deadline. A malformed or flattering `ActionDescription` corrupts approval, audit, and (in the proposed extension) attestation simultaneously.

**Simulation fidelity is a soft floor.** Where simulation has gaps, either the agent behaves incoherently or the gaps must be documented — at which point the approval mechanism leaks back into the agent's world-model. Outcome reconciliation (Section 6) partially *audits* simulation fidelity as a side effect, which is an additional argument for building it.

**Ephemerality is a discipline, not a property.** History's lesson from the spreadsheet era is that expressions of intention refuse to stay ephemeral: someone's throwaway Gadget becomes the artifact the quarterly close depends on. The bet that AI changes maintenance — the agent that wrote it can fix it; Blueprints re-derive rather than inherit — is credible and unproven over institutional timescales.

**The oracle problem bounds the whole edifice.** Epistemic authority and observer verification extend only as far as upstream ACL oracles exist. A System of Agency layered over Systems of Record that cannot answer "could this person see this?" degrades, by design, to all-or-nothing. Progress here is partly out of Cloudflare's hands — it is a demand on the record layer.

**The accounting boundary.** Model providers see the tokens; agent context is an unlabeled channel; cross-workspace flows through the model's own weights and the user's own head are outside the ledger. Perfect epistemic accounting is not on offer; the claim defended here is only that boundary-level accounting is sound, useful, and unprecedented at this fidelity — not that it is total.

---

## 8. Conclusion

Four connections, one claim.

Cloudflare OS collapses the boundaries between user and developer, application and document, human and process — and the collapse is only viable because the system first rebuilt trust: capabilities as mechanized institutional delegation (the authority passport), and information-flow tracking as the mechanized insider list (the knowledge passport). Between them stands the system's most elegant construction: a boundary at which an artificial actor's imagination is deliberately made indistinguishable from reality *to the actor*, precisely so that the distinction can be preserved *for the institution* — the agent as transaction, the human as commit.

What the system does not yet possess is the third passport: no primitive ever verifies that the promised condition became true. The commit phase returns `void`; promises are prose; revert is regret. The System of Agency has a front office and a middle office and no back office — every trade executes, none settles.

But the missing organ is latent in the built ones. The simulation machinery already manufactures machine-executable specifications of intent; retaining them across the commit boundary and reconciling them against the realized world turns the audit log into an attestation stream, divergence into a first-class event, and — once attestations become portable — the agent's history into a loss history. At that point the actor is not merely governable but underwritable, and trust in artificial actors stops being a policy problem inside one institution and becomes a priceable commodity in a market between them.

Systems of Record preserve what is true. Systems of Agency manage what can become true. The systems that verify **what actually became true, as promised** — the assurance layer — remain unbuilt.

That is the territory.

---

## Sources

All citations refer to the `cloudflare-os` repository (v2, August 2026 release).

1. `README.md` — product overview; "operating system" framing; office-suite analogy; "the centralized model of software stops making sense"; OS-analogy table (kernel = `packages/workshop-backend`, drivers = `packages/gatekeeper-*`).
2. `.agents/skills/write-gatekeeper/SKILL.md` — the seven Gatekeeper responsibilities; observation/action split (`authorizeObservation()` / `submitAction()` / `applyAction()`); "the most important and delicate part"; mandatory API-review stop; simulation requirement and the prohibition on documenting the approval queue to agents ("correct simulation keeps this invisible"); observer-verification strategies A–D.
3. `docs/observers.md` and `docs/sharing.md` — observer tracking and read-through sharing permissions; `addObserver` re-verification on every open; forward exclusion via `excludeObservers` (observation *blocked* while a named observer remains authorized); `prohibitAllSharing` lockdown (no sharing, no actions, no web fetches); per-gatekeeper strategy decision table including oracle-absent degradations (Home Assistant, ZoomInfo); capability-based role objects (`OverseerClientInterface` vs `UseOverseerInterface`).
4. `packages/workshop-shared/src/gatekeeper.ts` — `applyAction(action): Promise<void>`; `ActionDescription` JSDoc (approval routing, approver display, audit log; natural-language Markdown description); `revertAction()` semantics (best-effort, may fail, `implementsRevert` flag).

External traditions referenced: object-capability security (Dennis & Van Horn 1966; Miller's E; Cap'n Proto; Sandstorm); decentralized information flow control (Myers & Liskov 1997; Asbestos, HiStar, Flume); maker-checker and MNPI/insider-list regimes in financial-services governance; assurance as the audit profession's term for independent verification of representations.
