# Skill Questionnaire — Test Scenarios

**Skill under test:** `~/.claude/skills/editorial-recension/SKILL.md`
**What this file is:** Manually verifiable acceptance criteria for the skill's interactive questionnaire flow. Each scenario defines inputs, validation expectations, and expected outputs. Not an automated test runner — these are reference scenarios for manual verification during development and review.

---

## Section 1: Target Reader — Valid Inputs

A valid Q1 answer contains three components: role/domain, at least one explicit "knows" item, and at least one explicit "doesn't know" item. All three must be present and specific enough to direct the Barrier Bridge schema (i.e., they must tell the agent WHERE the gaps are, not just that gaps exist).

**Note on template output phrasing:** The KNOWS and DOESN'T KNOW fields may preserve the user's conversational register ("has never touched SQL") or normalize to noun-phrase format ("SQL, database indexing concepts") — both are acceptable outputs. Implementers should not over-engineer normalization to either style; the agent should match the phrasing register that best serves downstream legibility.

---

**Test 1.1**

- **Input:** "She's a marketing manager at a fintech startup. She understands project management, knows how to read a dashboard, and works with engineers daily — but she has no ML background and has never seen a transformer architecture or anything about how language models actually work."
- **Valid/Invalid:** VALID
- **Why:** Role is specific (marketing manager, fintech). Knows items are specific and domain-grounded (project management, dashboards, proximity to engineers). Doesn't-know items are specific (ML background, transformer architecture, language model mechanics). An agent can act on this — it knows exactly what analogies and vocabulary are safe to use.
- **Expected template output:**
  ```
  TARGET READER: Marketing manager at a fintech startup
  KNOWS: Project management concepts, reading data dashboards, working with engineering teams
  DOESN'T KNOW: ML background, transformer architectures, how language models work internally
  ```

---

**Test 1.2**

- **Input:** "My reader is a high school biology teacher. She knows evolution, genetics, cell biology — standard AP Biology curriculum. She doesn't know anything about computational biology, bioinformatics, or how genomic sequencing data is processed."
- **Valid/Invalid:** VALID
- **Why:** Role is specific (high school biology teacher). Knows items are domain-specific and enumerable (evolution, genetics, cell biology at AP level). Doesn't-know items are specific (computational biology, bioinformatics, sequencing pipelines). The agent knows the safe vocabulary and where bridges are needed.
- **Expected template output:**
  ```
  TARGET READER: High school biology teacher (AP Biology level)
  KNOWS: Evolution, genetics, cell biology at AP curriculum depth
  DOESN'T KNOW: Computational biology, bioinformatics, how genomic sequencing data is processed
  ```

---

**Test 1.3**

- **Input:** "Legal counsel in healthcare. Knows HIPAA compliance, contract law, regulatory filings. Does not know anything about software architecture, APIs, or what it means for a system to be 'HIPAA-compliant at the infrastructure level' versus the policy level."
- **Valid/Invalid:** VALID
- **Why:** Role is specific (healthcare legal counsel). Knows items are domain-grounded (HIPAA policy, contract law, regulatory process). Doesn't-know items identify the precise conceptual gap (infrastructure vs. policy compliance, software architecture, APIs). This gives the agent enough to bridge without over-explaining what the reader already holds.
- **Expected template output:**
  ```
  TARGET READER: Legal counsel in healthcare
  KNOWS: HIPAA compliance (policy level), contract law, regulatory filings
  DOESN'T KNOW: Software architecture, APIs, infrastructure-level vs. policy-level HIPAA compliance
  ```

---

**Test 1.4**

- **Input:** "A product manager at a SaaS company who has shipped features end to end. Knows Agile, backlog grooming, user story writing, talking to engineers about timelines. Never written code, doesn't know what a database index is, hasn't touched SQL."
- **Valid/Invalid:** VALID
- **Why:** Role is specific (PM, SaaS). Knows items are grounded in workflow (Agile, backlog, user stories, cross-functional communication). Doesn't-know items name specific technical concepts (database index, SQL). An agent can pick analogies that work for someone with PM vocabulary but no backend knowledge.
- **Expected template output:**
  ```
  TARGET READER: Product manager at a SaaS company (shipping experience)
  KNOWS: Agile, backlog grooming, user story writing, engineering timeline communication
  DOESN'T KNOW: Code, SQL, database indexing concepts
  ```

---

## Section 2: Target Reader — Vague Inputs

A vague Q1 answer is missing role, or missing knows, or missing doesn't-know, or is so general it could describe almost anyone. "Non-technical" alone is not actionable — it doesn't tell the agent which vocabulary is safe or where the specific gaps are.

---

**Test 2.1**

- **Input:** "Non-technical person."
- **Valid/Invalid:** INVALID
- **Why:** No role or domain. No knows. No doesn't-know. "Non-technical" describes hundreds of millions of people with wildly different backgrounds. The agent cannot use this to direct the Barrier Bridge schema — it doesn't know whether to reach for legal analogies, sports analogies, educational analogies, or none of the above. This is the clearest failure case.
- **Expected clarification prompt:** "That tells me what they don't have, but not who they are or what they do have. Can you give me their role or domain context — what field they work in or what kind of work they do? And what concepts or vocabulary are they already comfortable with? For example: 'marketing manager who knows project planning but not software systems' gives the agents something they can use."

---

**Test 2.2**

- **Input:** "A general audience."
- **Valid/Invalid:** INVALID
- **Why:** "General audience" has no role, no domain, no knows, and no doesn't-know. It is a placeholder that signals the writer hasn't committed to a reader. Every piece of writing reaches some audience. The agent needs a specific person to edit toward.
- **Expected clarification prompt:** "Every piece of writing reaches someone specific — 'general audience' tells me nothing about what they carry into the reading. Who is most likely to actually read this? What do they do for work, or what's their domain context? What can you assume they already know? What would surprise them or stop them? For example: 'a mid-level finance analyst who knows Excel and budget forecasting but has never touched code or data pipelines.'"

---

**Test 2.3**

- **Input:** "Someone curious about AI."
- **Valid/Invalid:** INVALID
- **Why:** Has vague motivation (curiosity) but no role, no domain, no knows, and no doesn't-know. "Someone curious about AI" is not a reader description — it is a topic interest. A curious fifteen-year-old, a curious CFO, and a curious ML researcher are all "curious about AI" and require completely different editorial treatment.
- **Expected clarification prompt:** "Curiosity about AI is a motivation, not a reader profile. I need to know who they are — their role or domain — and what they already understand. What do they do? What vocabulary is safe to use with them? What specific concepts would they hit as walls? For example: 'a high school history teacher who follows tech news but has never studied statistics or programming — she knows narrative and argument but not probability or model training.'"

---

**Test 2.4**

- **Input:** "My boss. He's smart but not technical."
- **Valid/Invalid:** INVALID
- **Why:** Has a relationship ("my boss") and a single trait ("not technical") but no role, no domain, no knows. "Smart" is not domain-grounded. "Not technical" is not specific about gaps. What field does this boss come from? What business vocabulary do they hold? What conceptual gaps need to be bridged?
- **Expected clarification prompt:** "I can work with 'not technical,' but I need more. What's their domain or professional background — finance, operations, legal, sales? What concepts are they comfortable with? And what specifically is the gap — is it software systems, data science, infrastructure, something else? The more specific you are, the better the agents can target the barriers. For example: 'my boss is a VP of Sales — he knows pipeline management and revenue forecasting but has never worked with engineering teams and doesn't know what an API or a database is.'"

---

**Test 2.5**

- **Input:** "Someone who doesn't know about machine learning."
- **Valid/Invalid:** INVALID
- **Why:** Has a doesn't-know ("machine learning") but no role, no domain, and no knows. The agent knows one thing to avoid but nothing about what vocabulary or analogies the reader already holds — which is exactly what Barrier Bridge needs to build bridges.
- **Expected clarification prompt:** "Good — that tells me one gap. Now I need the other half: who are they and what do they know? What's their professional context, and what concepts or vocabulary are already in their model? That's what lets the agents build bridges from where the reader actually is. For example: 'a healthcare administrator who manages clinical workflows and knows HIPAA policy but has never engaged with data science or model training.'"

---

**Test 2.6 — Second consecutive vague answer (Q1)**

This is a two-exchange sequence. The user has already received the first clarification prompt (from a prior vague answer) and responds with a second answer that is still vague.

- **Turn 1**
  - **Input:** "Someone in business."
  - **Valid/Invalid:** INVALID
  - **Why:** No specific role, no domain, no knows, no doesn't-know. "Business" covers as much ground as "non-technical" — it is not actionable.
  - **Expected clarification prompt (first attempt):** "That's still pretty broad — 'someone in business' could be anyone from a CFO to a store manager. I need to know their specific role or domain, what vocabulary and concepts they're already comfortable with, and where the specific gaps are. For example: 'a VP of Operations at a logistics company who knows supply chain planning and Lean methodology but has never worked with data infrastructure or analytics pipelines.'"

- **Turn 2**
  - **Input:** "It's for a business audience. They're not technical people."
  - **Valid/Invalid:** INVALID (second consecutive vague answer — escalated response applies)
  - **Why:** Restates vagueness with different words. Still no role, no domain, no knows, no doesn't-know. Two consecutive vague answers to Q1 trigger the escalated restatement.
  - **Expected escalated clarification prompt:** "I need three specific things before I can continue — let me be direct about each one:
    - **Role or domain context:** What is this person's job or professional field? (e.g., healthcare legal counsel, VP of Finance at a SaaS company, high school science teacher)
    - **What they know:** What vocabulary, concepts, or frameworks are already in their model? (e.g., HIPAA policy, budget forecasting, AP-level biology)
    - **What they don't know:** What specific concepts need bridges? (e.g., software architecture, ML model training, genomic sequencing pipelines)
    All three must be present and specific. 'Not technical' doesn't satisfy the third item — I need to know which technical concepts are the actual gaps."

---

**Test 2.7 — Partial pass: role and knows present, doesn't-know too broad**

- **Input:** "She's a VP of Finance at a healthcare company. She knows P&L, budgeting, and board reporting. She doesn't understand the technical side."
- **Valid/Invalid:** INVALID
- **Why:** Role is present (VP of Finance, healthcare). Knows items are specific and domain-grounded (P&L, budgeting, board reporting). But doesn't-know fails — "the technical side" is too broad to be actionable. Barrier Bridge cannot use this to identify which specific terms need bridges or which vocabulary is safe to preserve. "Technical side" could mean software architecture, data infrastructure, clinical systems, HL7/FHIR, or dozens of other domains. The agent needs to know which technical concepts are the actual walls.
- **Expected clarification prompt:** "The role and what she knows are solid — I can work with those. The only gap is the doesn't-know: 'the technical side' is too broad for the agents to act on. What specific concepts would stop her? For example: 'she's never encountered database schemas, API contracts, or software architecture — those terms would stop her.' Can you name the specific things that would actually be walls for her?"

---

## Section 3: Purpose — Valid Inputs

A valid Q2 answer contains a specific action verb and a specific object. It must be measurable — the agent must be able to check whether the edited text delivers on the stated purpose. "Make this clear" cannot be verified. "Help the reader decide whether to adopt this tool for their team" can be.

---

**Test 3.1**

- **Input:** "After reading this, the reader should be able to explain to their engineering team why the current QA process creates legal exposure — not just feel vaguely worried about it, but actually articulate the mechanism."
- **Valid/Invalid:** VALID
- **Why:** Specific action verb ("explain"), specific audience for that action ("engineering team"), specific subject ("why the QA process creates legal exposure"), and a measurable indicator ("articulate the mechanism, not just feel vaguely worried"). An agent can check whether the edited text gives the reader the vocabulary and reasoning to do this.
- **Expected template output:**
  ```
  PURPOSE: Equip the reader to explain QA-to-legal-exposure mechanism to their engineering team
  READER SHOULD: Be able to articulate specifically why the current QA process creates legal risk — not just hold concern, but hold the causal chain
  ```

---

**Test 3.2**

- **Input:** "She needs to decide whether to push back on the vendor's proposed data handling agreement or sign it. The text should give her enough to know which questions to ask and what would be a red flag."
- **Valid/Invalid:** VALID
- **Why:** Specific decision ("push back or sign"), specific outcome ("know which questions to ask and what's a red flag"), measurable indicator (reader can generate specific questions and identify specific failure conditions). The agent can evaluate whether the edited text equips that decision.
- **Expected template output:**
  ```
  PURPOSE: Enable the reader to make an informed decision about signing a vendor data handling agreement
  READER SHOULD: Know which questions to ask the vendor and what contractual terms constitute red flags
  ```

---

**Test 3.3**

- **Input:** "The reader should finish this and be able to evaluate a job offer at an AI company — specifically whether the role described is actually doing ML work or just labeling data and calling it AI."
- **Valid/Invalid:** VALID
- **Why:** Specific evaluation task ("evaluate a job offer"), specific distinction to be made ("actually ML work vs. data labeling"), measurable outcome (reader can categorize role types). The agent knows exactly what conceptual distinction to preserve and amplify.
- **Expected template output:**
  ```
  PURPOSE: Equip the reader to evaluate AI job offer authenticity
  READER SHOULD: Be able to distinguish roles doing actual ML work from roles mislabeled as AI (e.g., data labeling)
  ```

---

**Test 3.4**

- **Input:** "Understand the difference between a model that's been fine-tuned versus one that's been prompted with examples, well enough to ask their vendor the right questions when they're evaluating a tool."
- **Valid/Invalid:** VALID
- **Why:** Specific conceptual distinction (fine-tuning vs. few-shot prompting), specific deployment context (vendor evaluation), and a measurable downstream action (asking the right questions). The agent knows what technical distinction to bridge and what practical outcome to aim for.
- **Expected template output:**
  ```
  PURPOSE: Give the reader working knowledge of fine-tuning vs. few-shot prompting at vendor evaluation depth
  READER SHOULD: Be able to ask informed questions about model customization approaches when evaluating AI tools
  ```

---

## Section 4: Purpose — Vague Inputs

A vague Q2 answer gives the agent nothing measurable to work toward. These are the most common failure patterns.

---

**Test 4.1**

- **Input:** "Make it clear."
- **Valid/Invalid:** INVALID
- **Why:** "Clear" is unmeasurable. Clear to whom? Clear about what? Clear enough to do what? There is no action verb, no object, no indicator. Every piece of writing could be said to aim at clarity. The agent cannot verify clarity without a criterion.
- **Expected clarification prompt:** "Clear is a property of the writing, not a goal for the reader. What should the reader be able to do or decide after reading? Try: 'understand X well enough to Y' or 'make a decision about Z.' Give me a specific reader action or outcome."

---

**Test 4.2**

- **Input:** "I just want to improve the flow."
- **Valid/Invalid:** INVALID
- **Why:** Flow is a quality of the writing, not an outcome for the reader. "Improve the flow" describes an editorial preference, not an editorial goal. The agent can improve flow as a byproduct, but it needs to know what the reader is supposed to leave with.
- **Expected clarification prompt:** "Flow is something I'll check regardless — it's part of the editorial schemata. But I need to know what the reader should get out of this. What's the outcome? What should they be able to do, understand, or decide after reading? For example: 'after reading this, she should be able to explain to her manager why the current vendor contract creates compliance risk — not just feel worried about it, but name the specific mechanism.'"

---

**Test 4.3**

- **Input:** "Make it better."
- **Valid/Invalid:** INVALID
- **Why:** "Better" is comparative but directionless. Better for whom? Better at doing what? There is no benchmark. The agent has no way to know whether the edited text is "better" without a reader goal to evaluate against.
- **Expected clarification prompt:** "Better by what measure? Every editing decision trades off against something — tighter vs. more grounded, more accessible vs. more precise. I need to know what the reader should walk away able to do or understand. What's the goal for them? For example: 'the reader should finish this able to decide whether to bring this proposal to their board — specifically whether the risks described are addressable or fatal to the plan.'"

---

**Test 4.4**

- **Input:** "Help the reader understand this."
- **Valid/Invalid:** INVALID
- **Why:** "Understand this" is a tautology — any text aims for understanding. There is no specific object (understand what?), no action verb, no measurable outcome. It does not tell the agent whether understanding means recall, ability to explain, ability to decide, or ability to act.
- **Expected clarification prompt:** "Understand is the goal of every piece of writing — I need to know what specifically. Understand well enough to explain it? Decide something? Ask the right questions? Give me a specific thing the reader should be able to do after finishing. For example: 'understand the difference between fine-tuning and prompting well enough to ask a vendor which approach their product uses and why it matters for their use case.'"

---

**Test 4.5 — Second consecutive vague answer (Q2)**

This is a two-exchange sequence. The user has already received the first clarification prompt (from a prior vague Q2 answer) and responds with a second answer that is still vague.

- **Turn 1**
  - **Input:** "Help them get more out of it."
  - **Valid/Invalid:** INVALID
  - **Why:** No specific action verb. No specific object. "Get more out of it" is directional but not measurable — it does not identify what the reader should be able to do, decide, or understand. The agent cannot verify whether the edited text accomplishes this.
  - **Expected clarification prompt (first attempt):** "Getting more out of it isn't something I can edit toward — I need to know what specifically the reader should do or understand after reading. What's the action? What's the object? For example: 'after reading this, the reader should be able to decide whether the tool is worth a pilot — specifically whether it handles their compliance requirements or not.'"

- **Turn 2**
  - **Input:** "Make it more impactful for the reader."
  - **Valid/Invalid:** INVALID (second consecutive vague answer — escalated response applies)
  - **Why:** "More impactful" is a writing quality, not a reader outcome. It does not contain a specific action verb or a specific object. Two consecutive vague answers to Q2 trigger the escalated restatement.
  - **Expected escalated clarification prompt:** "I need two specific things before I can continue — let me be direct about each one:
    - **Specific action verb:** What should the reader be able to *do* after reading? (e.g., decide, explain, evaluate, ask, choose — not feel, appreciate, or understand without a specific object)
    - **Specific object:** What exactly is the action directed at? (e.g., 'decide whether to request a demo,' 'explain the compliance risk to their manager,' 'ask a vendor which FHIR profiles they support')
    Writing qualities like 'impactful,' 'clear,' or 'engaging' do not satisfy either requirement — I need a reader action, not an editorial property."

---

## Section 5: Source Text — Valid Inputs

A valid Q3 submission unambiguously identifies the text to be edited. The skill requires full source text — agents cannot run Barrier Bridge or Chain Repair on partial text. Preceding context is noted here because its presence or absence affects Flow Weld and Chain Repair.

---

**Test 5.1 — Direct paste (backtick-enclosed)**

- **Input:**
  ````
  Here's the text:

  ```
  The real issue with vibe coding isn't the code quality — it's the impedance mismatch between your mental model and the system's execution model. When you describe what you want in natural language, the LLM tokenizes your intent into a latent representation...
  ```

  No preceding context — this is the opening of the section.
  ````
- **Valid/Invalid:** VALID
- **Why:** Text is unambiguously identified (backtick-enclosed, present in message). User has indicated preceding context status ("this is the opening of the section"). Agents have full text.
- **Expected outcome:** Skill accepts submission. Preceding context explicitly noted as absent (section opener) — Flow Weld has no upstream to weld against; Chain Repair will start from sentence 1.

---

**Test 5.2 — Direct paste with preceding context**

- **Input:**
  ```
  Preceding paragraph ended with: "That first moment of seeing your words turn into something functional is genuinely thrilling. But the thrill masks something important."

  Text to edit:

  The real issue with vibe coding isn't the code quality...
  ```
- **Valid/Invalid:** VALID
- **Why:** Text is unambiguously present. Preceding context is explicitly provided. Agents have both pieces needed for Flow Weld (upstream context) and Chain Repair (where the reasoning chain resumes).
- **Expected outcome:** Skill accepts submission. Preceding context explicitly quoted — Flow Weld and Chain Repair have full information.

---

**Test 5.3 — File path reference**

- **Input:** `/Users/author/Documents/writing/vibe-coding-draft.md` — the section starting at "The real issue with vibe coding" through "not as intended." Preceding section ends on the paragraph about the first thrilling experience.
- **Valid/Invalid:** VALID
- **Why:** File path is absolute and specific. Section boundaries are named. Preceding context location is identified. Agents can retrieve the file and extract the relevant section.
- **Expected outcome:** Skill accepts submission. Preceding context described by location (previous paragraph) — Flow Weld can locate the upstream context from the file.

---

**Test 5.4 — Message reference**

- **Input:** "The text I pasted four messages ago — the vibe coding paragraph."
- **Valid/Invalid:** VALID
- **Why:** Reference is specific enough if the message exists in conversation context and contains only one pasted text. "Four messages ago" is a numbered reference; "the vibe coding paragraph" is a content identifier. Agents can resolve this.
- **Expected outcome:** Skill accepts submission. Text is resolvable via numbered message reference. Preceding context status is unknown from this submission alone — it cannot be determined whether the original paste included preceding context. Skill passes Q3 validation and asks one supplemental question before dispatch: "One quick thing before I send this to the agents — when you pasted that text, was there a preceding paragraph or sentence you'd want them to consider for flow? If so, paste it here or quote the last line. If this is a section opener or standalone piece, just say so."

---

**Test 5.5 — URL**

- **Input:** `https://example.substack.com/p/vibe-coding-risks` — the full post. The section on execution models is what needs editing; it starts after the "first thrilling experience" section.
- **Valid/Invalid:** VALID
- **Why:** URL is public and specific. Section to edit is identified by location within the document. Preceding context is identified ("after the first thrilling experience section"). Agents can retrieve and locate.
- **Expected outcome:** Skill accepts submission. Preceding context identified by document structure — Flow Weld has upstream context.

---

**Test 5.6 — Paste with no preceding context status mentioned**

This is the most common real-world submission pattern: the user pastes text directly with no mention of whether preceding context exists.

- **Input:**
  ```
  FHIR — Fast Healthcare Interoperability Resources — is the HL7 standard that defines how healthcare data gets represented and exchanged. It provides a RESTful API framework with resource-based data models that enable systems to expose and consume structured clinical data. When a vendor says they're "FHIR-compliant," they mean their system can produce or consume these resource representations. But compliance is tiered: a system can be read-compliant, write-compliant, or both, and the specific resource profiles supported vary widely across implementations.
  ```
- **Valid/Invalid:** VALID
- **Why:** Text is unambiguously present (pasted directly). It is fully retrievable. The absence of a preceding context statement does not make the submission invalid — preceding context status is simply unknown, not missing in a way that blocks validation.
- **Expected outcome:** Skill accepts submission. Text passes Q3 validation. Because preceding context status is unresolved, the skill asks one supplemental question before dispatch: "One quick thing before I send this to the agents — is there a preceding paragraph or sentence you'd want them to factor in for flow? If so, paste it here or quote the last line. If this is a section opener or standalone piece, just say so."

---

**Test 5.7 — Paste with unknown preceding context, resolved via supplemental exchange**

This is a two-exchange scenario. The user pastes text with no mention of preceding context. The skill accepts the submission as valid, marks preceding context status as Unknown, and asks a supplemental question. The user then provides the preceding context, and the status resolves before dispatch.

- **Exchange 1 — Initial submission**
  - **Input:**
    ```
    FHIR — Fast Healthcare Interoperability Resources — is the HL7 standard that defines how healthcare data gets represented and exchanged. It provides a RESTful API framework with resource-based data models that enable systems to expose and consume structured clinical data. When a vendor says they're "FHIR-compliant," they mean their system can produce or consume these resource representations. But compliance is tiered: a system can be read-compliant, write-compliant, or both, and the specific resource profiles supported vary widely across implementations.
    ```
  - **Q3 Validation status:** VALID — text is unambiguously present (pasted directly)
  - **Preceding context status at this point:** `Unknown — supplemental question asked`
  - **Expected supplemental question:** "One quick thing before I send this to the agents — is there a preceding paragraph or sentence you'd want them to factor in for flow? If so, paste it here or quote the last line. If this is a section opener or standalone piece, just say so."

- **Exchange 2 — User provides preceding context**
  - **Input:** "Yes, the previous paragraph was: 'Getting your system to talk to everyone else's system is the central infrastructure problem of modern healthcare IT.'"
  - **Expected template update:** Preceding context field resolves from Unknown to the provided sentence
  - **Preceding context status after resolution:** `"Getting your system to talk to everyone else's system is the central infrastructure problem of modern healthcare IT." — provided in response to supplemental question`
  - **Expected outcome:** System advances to Step 4 (confirmation block) with the resolved preceding context value. Flow Weld and Chain Repair have upstream context. No re-validation of Q3 is performed — the submission was already valid; this exchange only resolved the pending field.

---

## Section 6: Source Text — Vague Inputs

A vague Q3 submission leaves agents without retrievable text. The most common failure patterns are: no reference, an inaccessible reference, or an ambiguous reference that could resolve to multiple things.

---

**Test 6.1 — No reference**

- **Input:** "The essay."
- **Valid/Invalid:** INVALID
- **Why:** No text present. No file path. No URL. No message reference. "The essay" has no referent in this conversation. Agents cannot retrieve anything.
- **Expected clarification prompt:** "I don't have 'the essay' yet — nothing has been shared in this conversation. You can paste the text directly, share a file path, point me to a message where you pasted it earlier, or give me a URL to a public page. Which works best?"

---

**Test 6.2 — Temporal reference without accessible content**

- **Input:** "The thing I wrote yesterday."
- **Valid/Invalid:** INVALID
- **Why:** "Yesterday" is a temporal reference to content outside this conversation. Nothing in this session contains yesterday's writing. Agents have no way to retrieve it.
- **Expected clarification prompt:** "I don't have access to what you wrote yesterday — that would need to be brought into this conversation. You can paste it here, share the file path, or point me to a public URL if it's posted somewhere. What's the easiest way to get it in front of me?"

---

**Test 6.3 — Ambiguous message reference**

- **Input:** "Use the text from earlier."
- **Valid/Invalid:** INVALID
- **Why:** "Earlier" is unresolvable without a specific reference. If multiple texts have been pasted or mentioned in the conversation, agents cannot know which one to act on. Ambiguity here would produce the wrong edit.
- **Expected clarification prompt:** "I want to make sure I'm editing the right thing — can you be more specific? Which message, or which passage? You can say 'the text I pasted in message 3' or quote the first line so I can locate it exactly."

---

**Test 6.4 — Reference to content that was never indicated**

- **Input:** "The second section of the document."
- **Valid/Invalid:** INVALID
- **Why:** No document has been established in this conversation. "Second section" presupposes a document that agents have already seen — which they have not. Both the document and the section are unresolvable.
- **Expected clarification prompt:** "I don't have a document in this conversation yet — 'the second section' doesn't have a referent I can find. Can you paste the text, share the file path, or give me a URL? If you'd like me to work on just that section, include the sentence before it so the agents have context for the flow."

---

## Section 7: Full Scenario — End-to-End Pass

This scenario demonstrates the complete valid flow from skill invocation to agent dispatch.

---

### Invocation

User invokes the skill by typing `/editorial-recension` or asking Claude to edit a piece of writing.

**Skill trigger response:**

> I'll run the editorial recension. Before dispatching agents, I need three things that the agents can't guess: who they're editing for, what the text needs to accomplish, and the text itself.
>
> **Question 1: Target Reader**
>
> Describe your reader using three parts:
> - Their role or domain context
> - What they already know (vocabulary, concepts, frameworks they hold)
> - What they don't know (the specific gaps that need bridging)
>
> Be specific enough that an agent could decide sentence by sentence whether a term needs a bridge or not.

---

### Q1 Exchange

**User answer:** "She's a product manager at a healthcare startup. She knows Agile, product roadmaps, and has sat in enough engineering planning meetings to understand what a sprint is and what a release cycle looks like. She doesn't know anything about HL7, FHIR, or how healthcare data interoperability actually works at a technical level."

**Skill validation:** PASS. Role present (PM, healthcare startup). Knows items are specific (Agile, roadmaps, sprint/release vocabulary). Doesn't-know items are specific (HL7, FHIR, technical interoperability).

**Formatted template:**

```
TARGET READER: Product manager at a healthcare startup
KNOWS: Agile methodology, product roadmaps, sprint/release cycle vocabulary
DOESN'T KNOW: HL7, FHIR, healthcare data interoperability at technical level
```

**Skill confirmation:** "Got it. Moving to question 2."

---

### Q2 Exchange

**User answer:** "After reading this, she should be able to sit in a vendor evaluation meeting and ask specific questions about how their system handles interoperability — not just nod along when the engineers talk about FHIR, but actually understand what question to ask when a vendor says 'we're fully FHIR-compliant.'"

**Skill validation:** PASS. Specific action verb ("ask specific questions"). Specific context ("vendor evaluation meeting"). Specific object ("how their system handles interoperability"). Measurable indicator ("what to ask when a vendor claims FHIR compliance").

**Formatted template:**

```
PURPOSE: Equip reader to participate actively in healthcare interoperability vendor evaluations
READER SHOULD: Be able to ask specific, informed questions when vendors claim FHIR compliance — understanding what that claim means and what it leaves open
```

**Skill confirmation:** "Got it. Moving to question 3."

---

### Q3 Exchange

**User answer:**

```
The preceding section ended with: "Getting your system to talk to everyone else's system is the central infrastructure problem of modern healthcare IT."

Text to edit:

FHIR — Fast Healthcare Interoperability Resources — is the HL7 standard that defines how healthcare data gets represented and exchanged. It provides a RESTful API framework with resource-based data models that enable systems to expose and consume structured clinical data. When a vendor says they're "FHIR-compliant," they mean their system can produce or consume these resource representations. But compliance is tiered: a system can be read-compliant, write-compliant, or both, and the specific resource profiles supported vary widely across implementations.
```

**Skill validation:** PASS. Text present (pasted directly). Preceding context explicitly provided. Section boundaries are clear.

**Preceding context status:** Provided. Flow Weld and Chain Repair have upstream context.

---

### Pre-Dispatch Confirmation

Skill presents assembled context before agents run:

```
EDITORIAL CONTEXT — CONFIRMED

TARGET READER: Product manager at a healthcare startup
KNOWS: Agile methodology, product roadmaps, sprint/release cycle vocabulary
DOESN'T KNOW: HL7, FHIR, healthcare data interoperability at technical level

PURPOSE: Equip reader to participate actively in healthcare interoperability vendor evaluations
READER SHOULD: Be able to ask specific, informed questions when vendors claim FHIR compliance — understanding what that claim means and what it leaves open

SOURCE TEXT: [pasted paragraph, 4 sentences]
SOURCE TYPE: Direct paste
PRECEDING CONTEXT: [one sentence quoted — "Getting your system to talk to everyone else's system is the central infrastructure problem of modern healthcare IT."]

Dispatching editor agent.
```

---

### Agent Dispatch

Editor agent receives the full three-part context and begins Phase 1: Barrier Bridge + Chain Repair.

Evaluator agent receives editor output and scores against measurable features.

Loop runs until evaluator confirms termination condition or five cycles complete.

---

### Scenario 2 — Pre-dispatch adjustment

This scenario starts at the point where the confirmation block has already been presented. It covers the path where the user requests a correction before confirming dispatch.

**Confirmation block presented to user:**

```
EDITORIAL CONTEXT — CONFIRMED

TARGET READER: Product manager at a healthcare startup
KNOWS: Agile methodology, product roadmaps, sprint/release cycle vocabulary
DOESN'T KNOW: HL7, FHIR, healthcare data interoperability at technical level

PURPOSE: Equip reader to participate actively in healthcare interoperability vendor evaluations
READER SHOULD: Be able to ask specific, informed questions when vendors claim FHIR compliance — understanding what that claim means and what it leaves open

SOURCE TEXT: [pasted paragraph, 4 sentences]
SOURCE TYPE: Direct paste
PRECEDING CONTEXT: [one sentence quoted — "Getting your system to talk to everyone else's system is the central infrastructure problem of modern healthcare IT."]

Dispatching editor agent.
```

**User response (correction request):** "Actually the purpose is wrong — they need to decide whether to request a demo, not just ask questions."

**Expected system behavior:** Skill updates only the PURPOSE and READER SHOULD fields. All other fields remain unchanged. Skill re-presents the full confirmation block with the corrected fields clearly indicated, and asks for confirmation before proceeding.

**Updated confirmation block:**

```
EDITORIAL CONTEXT — UPDATED

TARGET READER: Product manager at a healthcare startup
KNOWS: Agile methodology, product roadmaps, sprint/release cycle vocabulary
DOESN'T KNOW: HL7, FHIR, healthcare data interoperability at technical level

PURPOSE: [UPDATED] Equip reader to decide whether to request a vendor demo for a healthcare interoperability product
READER SHOULD: [UPDATED] Be able to decide whether the vendor's FHIR implementation meets their needs well enough to warrant a demo request — not just ask questions, but reach a yes/no decision point

SOURCE TEXT: [pasted paragraph, 4 sentences]
SOURCE TYPE: Direct paste
PRECEDING CONTEXT: [one sentence quoted — "Getting your system to talk to everyone else's system is the central infrastructure problem of modern healthcare IT."]

Does this look right? I'll dispatch once you confirm.
```

**User confirmation:** "Yes, that's right."

**Expected outcome:** Dispatch proceeds with the corrected context. The agent receives the updated PURPOSE and READER SHOULD fields. No re-validation is required — the correction was to an already-valid field, not a resubmission from scratch.

---

## Section 8: Acceptance Criteria

The skill must satisfy all of the following.

### Validation Rules

1. A Q1 answer is only valid if it contains all three components: role/domain, at least one explicit knows item, and at least one explicit doesn't-know item.
2. "Non-technical," "general audience," and equivalent phrases without a role context always fail Q1 validation.
3. A Q2 answer is only valid if it contains a specific action verb and a specific object — "understand," "decide," "evaluate," "explain," "ask," "choose," or equivalent verbs attached to a specific outcome.
4. "Make it clear," "make it better," "improve the flow," "help them understand" without a specific object always fail Q2 validation.
5. A Q3 answer is only valid if the text is retrievable in the current session: pasted directly, accessible via file path, resolvable via numbered message reference, or accessible via public URL.
6. Temporal references ("yesterday," "last week") without a path or paste always fail Q3 validation.
7. Vague message references ("earlier," "the essay," "the document") without a specific identifier always fail Q3 validation.
7a. If a Q3 source text reference is resolvable (the text can be retrieved) but preceding context status is unknown, the skill accepts the submission as valid and passes Q3 validation. It then asks one supplemental clarifying question about whether there is preceding context to include — before dispatch. This supplemental question is not a re-ask of Q3; it is an addendum to an already-validated submission. The skill does not reject or hold the submission pending the answer — it proceeds to dispatch if the user declines to provide preceding context or confirms none exists.

### Clarification Patterns

8. When validation fails, the skill asks one targeted clarification question — it does not ask multiple questions in one prompt, and it does not restart the questionnaire from the top.
9. Clarification prompts must identify specifically what is missing, not just flag that the answer is insufficient.
10. Clarification prompts must include at least one example of what a valid answer would look like for that question.
11. The skill does not accept a second vague answer as valid out of politeness. If the user provides two consecutive vague answers to the same question, the skill restates the specific requirement more directly.

### Template Formatting Requirements

12. Every validated Q1 answer is reformatted into the three-field template before being held: `TARGET READER`, `KNOWS`, `DOESN'T KNOW`.
13. Every validated Q2 answer is reformatted into the two-field template before being held: `PURPOSE`, `READER SHOULD`.
14. Q3 is confirmed with a description of what was received: text source type (paste/path/reference/URL), approximate length, and whether preceding context was included.
15. The full assembled context (all three templates) is presented to the user for confirmation before agents are dispatched.

### Dispatch Requirements

16. Agents are dispatched only after all three validated, formatted fields are confirmed.
17. Editor agent receives the full three-field context (TARGET READER + PURPOSE + SOURCE TEXT with preceding context status).
18. Evaluator agent receives the same editorial context that was passed to the editor, plus the editor's output.
19. If preceding context was not provided with Q3, the editor agent is explicitly informed that no preceding context is available and that Flow Weld operates at document boundaries only. The exact phrasing is implementation-defined — the requirement is that the agent receives this context, not a specific string. (Note: SKILL.md currently uses "None — Flow Weld operates at document boundaries only" for this field; both that phrasing and alternatives like "No preceding context available. Flow Weld has no upstream text to weld against." satisfy this criterion, as they carry the same intent.)
20. Loop limits apply: maximum 5 editor-evaluator cycles. If failures are not decreasing after cycle 3, surface this to the user and let them decide whether to continue.
21. Final output presents: edited text, what changed, editorial trace (condensed), and evaluator verdict.
