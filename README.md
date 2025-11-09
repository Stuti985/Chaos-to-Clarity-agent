# Chaos-to-Clarity-agent
An AI system that transforms unstructured user input into a clean, structured, and actionable plan using reasoning, memory, and iterative refinement.

SECTION 1: BASIC DETAILS

Name: Stuti Tripathi

AI Agent Title / Use Case: Chaos-to-Clarity Agent — converts messy, mixed, or emotionally-charged user inputs into a concise, structured interpretation and next-step suggestions.

SECTION 2: PROBLEM FRAMING

2.1. What problem does your AI Agent solve?

Many users express thoughts that are unstructured, emotionally mixed, or overwhelmed (“I’m tired but excited and I don’t know what to do”). The agent’s role is to convert such chaotic input into clear, bite-sized meaning: a distilled interpretation, detected emotions, core needs, and practical next steps.

2.2. Why is this agent useful?

It reduces cognitive load, improves clarity of communication, and helps users move from confusion to actionable perspective quickly. It is useful for journaling, prepping for conversations, clarifying goals, or composing clearer messages.

2.3. Who is the target user?

Students, early professionals, managers, or anyone who frequently writes informal, mixed-emotion messages and wants a fast structured interpretation to act on or reflect with.

2.4. What not to include?

- Not a replacement for professional therapy or medical advice.
- Will not make high-stakes decisions for users (e.g., financial/legal decisions).
- Will not store sensitive personal identifiers beyond session unless explicit consent & secure storage are implemented (out of scope for this assignment).

SECTION 3: 4-LAYER PROMPT DESIGN

Design principle: each prompt performs one responsibility only and returns a machine-parsable response (JSON or clearly delimited output) to avoid ambiguity.

3.1 INPUT UNDERSTANDING

Prompt (system / user prompt sent to LLM):

You are an Input Understanding module. Given the user's raw text, extract:
1) A short, formal "Restatement" (one sentence) that captures the user's main concern.
2) Detected emotions as a list (e.g., ["tired","excited","anxious"]).
3) Key themes or topics as a list (e.g., ["career","energy","decision"]).
4) Ambiguities or missing information (short note).
Return a JSON object: {"restatement": "...", "emotions": [...], "themes": [...], "ambiguities": "..."}
Respond only with the JSON.

Responsibility: Convert informal/chaotic input into a compact, structured representation for downstream components.

Example Input → Output (tested):
- Input: bro I’m tired but also excited but idk what to do with my life lol
- Output JSON:
 
{

  "restatement": "User feels conflicted: low energy yet anticipatory and uncertain about life direction.",
  
  "emotions": ["tired","excited","uncertain"],
  
  "themes": ["life direction","energy","motivation"],
  
  "ambiguities": "Unclear whether the user wants advice, reflection prompts, or concrete next steps."
  
}

3.2 STATE TRACKER

Prompt / Logic (internal module / system message):

Maintain session state as simple key-value variables:
- last_restements: [ ... ]
- last_themes: [ ... ]
- user_preferences: { tone: "formal|casual", depth: "brief|detailed" }

When generating suggestions, compare new themes to last_themes and avoid repeating identical suggestions in consecutive interactions.
If user consents, persist preferences across sessions.

- How it helps memory: Simulates short-term memory within a session and provides repeat-avoidance behavior. Memory is shallow and explicit (arrays/objects) to keep behavior auditable.
- Implementation note: For a prototype, store state in server memory (or browser localStorage) with session ID; in production use secure, consented storage.

3.3 TASK PLANNER

Prompt (instruction to internal planner logic or explicit LLM step):

Plan these internal steps:
1. Receive parsed JSON from Input Understanding.
2. Classify user intent among: {reflect, vent, ask_for_advice, ask_for_next_steps, unknown}.
3. If intent == unknown, prepare a clarifying question (one short sentence).
4. Otherwise, produce:
   - A concise "Meaning" sentence (1 line),
   - A ranked list of 3 practical next steps or question prompts,
   - A "tone" and "format" recommendation for replying to others (if user asked to reword).
5. Enforce constraints: outputs must be short, non-judgmental, and avoid medical/legal advice.
Return a structured plan object.

- Responsibility: Break the job into deterministic substeps (intent detection, meaning synthesis, actionable suggestions), controlling branching (clarify vs. suggest).
- Chaining/Branching: Uses a small chain: parse → intent → either clarify (branch) or produce recommended actions.

3.4 OUTPUT GENERATOR

Prompt (final content generator):

Using the planner output, produce a final response with three sections:
1) RESTATEMENT (1 sentence) — formal, clear.
2) WHAT I HEAR (bulleted emotions + themes).
3) NEXT STEPS (3 brief, prioritized actions or reflection prompts). 
Keep language professional, concise, and empathetic. Use markdown formatting. Maximum 120 words.
Respond in English.

- Formatting / Tone: Crisp, professional, empathetic. Markdown with bullets and small sentences to maximise scannability. Example output shown in Section 5.

SECTION 4: CHATGPT EXPLORATION LOG

Minimum three meaningful iterations. Documented as prompt variant → result → change → rationale.

| Attempt | Prompt Variant / Stage Tested | What Happened | What You Changed | Why You Changed It |
|--------|-------------------------------|----------------|------------------|---------------------|
| **1** | Basic Input Understanding (simple restatement) | Model either echoed input verbatim or produced long, unstructured paragraphs. | Forced a strict JSON schema with keys: meaning, emotions, themes, ambiguities. | To stop uncontrolled output and create a machine-readable foundation for the agent. |
| **2** | Added emotion extraction + ambiguity detection | Emotions were captured correctly, but planner generated long, unfocused suggestions. | Added Task Planner with intent classification (clarify/guide) + length limits. | To keep the output concise, intentional, and aligned with the agent’s workflow. |
| **3** | Introduced State Tracker (tracking last themes) | Repetition reduced, but sometimes removed useful context. | Updated rules: avoid identical repeats; allow useful continuity. | To balance memory with freshness in responses. |
| **4** | Output Generator with word limit + markdown | Output was clear but lacked consistent emotional framing. | Refined phrasing templates for empathy + professional tone. | To improve tone consistency and readability. |
| **5** | Edge case tests (empty / one word input) | Model struggled to infer intent. | Added fallback rule: if ambiguity high → ask one clarifying question. | Ensures graceful handling of vague inputs. |

SECTION 5: OUTPUT TESTS (examples)

Test 1 — Normal mixed-emotion input

- Input: bro I’m tired but also excited but idk what to do with my life lol
- Agent Output (final):
**RESTATEMENT**
You feel low on energy but also anticipatory, and uncertain about your direction in life.

**WHAT I HEAR**
- Emotions: tired, excited, uncertain
- Themes: life direction, motivation, mixed energy

**NEXT STEPS**
1. Spend 10 minutes listing activities that energize vs. drain you this week.
2. Identify one small experiment (2–3 hours) that explores something you’re excited about.
3. If you want clarity, answer: “What would I try if failure wasn’t a concern?” (one sentence).

- Notes: This output is concise (≤120 words), structured, and directly actionable.

Test 2 — Vague short input
- Input: idk
- Agent Output:
**RESTATEMENT**
You are unsure and seeking direction, but the specifics are unclear.

**WHAT I HEAR**
- Emotions: uncertain
- Themes: unspecified

**NEXT STEPS**
1. Can you tell me whether you want (A) reflection prompts, (B) quick actions, or (C) help drafting a message? (reply A/B/C)

- Notes: Agent asks a clarifying question when signal is low.

Test 3 — Empty input / noise
- Input: `` (empty)
- Agent Output:
I didn’t catch any content. Would you like a quick reflection prompt to get started? (Yes / No)

Explanation of the earlier “Raw Input == Raw Output” issue:

When the Input Understanding prompt was unconstrained, the model sometimes simply echoed the user text (mirror response) rather than extracting structure. Requiring a JSON response with explicit keys (restatement, emotions, themes, ambiguities) prevents raw echoing and ensures downstream modules receive structured data.

SECTION 6: REFLECTION

6.1. What was the hardest part of this assignment?

The hardest part was decomposing the agent into distinct, non-overlapping prompt responsibilities while ensuring information flows precisely between modules. Early iterations showed leakage: the generator would re-infer things already extracted by the Input Understanding layer. Designing machine-parsable outputs (JSON) and enforcing strict format constraints solved most leakage.

6.2. What part did you enjoy the most?

I enjoyed iterating prompts and watching how small structural constraints improved result quality dramatically. Constraining outputs to specific keys and short lengths produced much more reliable, testable behaviors. Tuning tone to be professional yet empathetic was satisfying.

6.3. If given more time, what would you improve or add?

I would implement persistent, consented user memory and a lightweight preference UI so the agent tailors depth and tone over multiple sessions. I would also add a small supervised classifier for intent to reduce ambiguity and integrate a feedback loop where users rate helpfulness to refine suggestions.

6.4. What did you learn about ChatGPT or prompt design?

Clear separation of concerns and machine-readable formats (JSON) dramatically reduce unintended behavior. Short, explicit instructions (e.g., “respond only with JSON”) are more reliable than broad directives. Iteration and small tests are essential to validate assumptions.

6.5. Did you ever feel stuck? How did you handle it?

Yes — the model initially echoed inputs or produced overlong replies. I handled this by constraining outputs, adding an ambiguity field, and creating a planner that enforced intent branching. Testing edge inputs and building fallback clarifying questions resolved most stuck states.

SECTION 7: HACK VALUE (Optional)
- Implemented a simple short-term memory simulation (last_themes, last_restements) to avoid immediate repetition and preserve helpful continuity.
- Designed explicit planner branching (clarify vs. suggest) to handle low-signal inputs gracefully.
- Included a compact integration pseudocode (below) to show how prompts chain and where system messages belong.
