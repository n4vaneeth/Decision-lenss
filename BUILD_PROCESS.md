# Build Process


##  Build Process Overview

| Required Explanation | Where It Is Covered |
|---------------------|---------------------|
| How I started | [Phase 1 – Initial Understanding & Static Prototype](#phase-1-initial-understanding--static-prototype) |
| How my thinking evolved | [Phases 2–5](#phase-2-design-evolution--refinement) |
| Alternative approaches considered | [Phases 2](#phase-2-design-evolution--refinement) & [4](#phase-4-critical-usability--model-flaw-discovery) |
| Refactoring decisions | [Phases 6–8](#phase-6-refactoring-validation--stability) |
| Mistakes and corrections | [Phases 4](#phase-4-critical-usability--model-flaw-discovery), [8](#phase-8-input-flow-corrections--ux-guardrails), [9](#phase-9-structure-agent-corrections--human-validation), and [10](#phase-10-ux-dead-ends-scale-editing--explanation-consistency) |
| What changed during development and why | [Phases 5–10](#phase-5-revised-model--ai-assisted-structuring) |

## Phase 1: Initial Understanding & Static Prototype
The goal was to build a Decision Companion System (DCS) that assists users in making better decisions by structuring trade-offs explicitly. The system was designed to prioritize deterministic logic over black-box AI recommendations to ensure transparency and trust.

### Approach:
- Defined the core data structures: options, criteria, weights, and ratings.
- Implemented the `compute_weighted_scores` engine using standard Python.
- Validated the logic using a static, hardcoded model in `main.py` to ensure mathematical accuracy before building the user interface.

## Phase 2: Design Evolution & Refinement
Initially, a chatbot-style interface was considered. However, this was rejected to avoid obscuring the decision logic and over-relying on AI. The design was refined to:
- Use deterministic weighted scoring as the core decision engine.
- Position AI strictly as an optional assistive layer.
- Ensure the system remains functional even if AI components are unavailable.

## Phase 3: Transition to Dynamic CLI
After validating the core engine, the focus shifted toward building a dynamic CLI-based system that allows real user input.

### Key Implementations
- **Interactive Input Layer:** Created `src/utils/input_helpers.py` to collect options and criteria dynamically.
- **Dynamic Weighting:** Allowed users to express importance, which the system normalized so that weights always sum to 1.
- **Validation Layer:** Added checks in `src/core/validation.py` to ensure completeness and consistency of inputs.

At this stage, the system became fully dynamic and runnable end-to-end.

## Phase 4: Critical Usability & Model Flaw Discovery

During real-world testing and iterative use, the system exposed fundamental modeling and usability flaws. These issues were not implementation bugs but structural problems in how decisions were being represented and evaluated.

### Cost vs Benefit Criteria Misinterpretation

The initial scoring logic implicitly treated all criteria as “higher is better.”

- This caused incorrect behavior for cost-based criteria (e.g., price, time, effort), where lower values should be preferred.
- Although criteria were labeled as cost or benefit at input time, this distinction was not properly integrated into normalization and scoring logic.
- As a result, the model could produce rankings that directly contradicted user intent.

This revealed that correct handling of cost vs benefit is not a UX concern but a core mathematical requirement of any decision model.

### User Cognitive Overload & Input Model Breakdown

The original interaction model required users to:
- Rank criteria by importance.
- Manually rate every option on a 1–10 scale for every criterion.

Testing showed this dual-input requirement to be unintuitive, error-prone, and cognitively unrealistic for most users.

Users consistently struggled to:
- Translate real-world facts into abstract numeric ratings.
- Understand how ratings interacted with priority weights.
- Predict or trust the resulting rankings.

Even when user intent was clear, outcomes often felt confusing or counterintuitive.
This demonstrated that the problem was not user error, but a flawed assumption that humans can reliably perform normalization and scoring tasks.

## Phase 5: Revised Model — AI-Assisted Structuring 

> Users should express intent and facts, not perform scoring or normalization.

### Revised Architecture

The system is now divided into three clearly defined layers:

1.  **Structure Agent (AI-Assisted)**
    - Converts natural language input into structured data.
    - **Extracts:**
        - Options
        - Criteria
        - Cost vs benefit classification
        - Option attributes (numeric or descriptive)
    - **Emits** ordinal scales for descriptive criteria.
    - **Marks** missing information explicitly.
    - **Does not:**
        - Score, rank, or assign weights.

2.  **Core Decision Engine (Deterministic)**
    - Converts ordinal descriptors to numeric values.
    - Accepts raw numeric attributes directly.
    - Normalizes values across options.
    - Applies user-defined criterion weights.
    - Correctly handles cost vs benefit criteria.
    - Produces ranked results.
    - No AI is used in this layer.

3.  **Explanation Agent (Optional AI)**
    - Explains rankings using engine outputs.
    - Highlights trade-offs based on user priorities.
    - **Does not** recalculate or override results.

### Attribute Model

The system supports:
-   Numeric attributes (e.g., price, calories, protein)
-   Ordinal attributes (e.g., cheap → expensive, low → high)

Ordinal scales are explicit and decision-specific.

### Missing-Data Policy

If an option lacks data for a criterion:
-   That criterion is excluded for the option.
-   Remaining weights are re-normalized.

This avoids penalizing uncertainty or encouraging guesses.

### Outcome

The revised model:
-   Removes arbitrary 1–10 ratings
-   Reduces user effort
-   Remains fully explainable and deterministic
-   Works across multiple decision domains

### Key Insight
> Users should express intent and factual information — not perform normalization, inversion, or scoring themselves. Normalization and trade-off math are system responsibilities, not human ones.

### Revised Interaction Model

**Users now:**
-   Define decision options.
-   Define evaluation criteria.
-   Rank criteria by importance.
-   Describe option attributes using natural language or factual terms.

**The system now:**
-   Explicitly classifies criteria as cost or benefit.
-   Normalizes attributes across options in a scale-independent manner.
-   Applies user-defined weights deterministically.
-   Produces a ranked outcome based on transparent calculations.

This preserves user control over priorities while removing unnecessary cognitive burden.

## Phase 6: Refactoring, Validation & Stability

As the system grew in complexity, multiple refactoring passes were required to restore clarity and correctness.

Key refactoring decisions included:
- Separated attribute conversion logic into a dedicated mapping layer
- Isolated normalization and scoring into a single deterministic engine
- Split explanation generation into:
  - a factual explanation payload
  - an optional AI-based natural language renderer
- Reworking input helpers to prevent silent failures and skipped user input

This phase focused less on adding features and more on making the system predictable, debuggable, and mathematically consistent.


## Phase 7: Stability, Missing Data Handling & User Control
As the system was exercised with real inputs, a major concern emerged around missing or incomplete attribute data. In real-world decisions, users often do not know every value for every criterion, and forcing guesses would reduce trust.

### Design Decision: Missing Data Policy
- If an option lacks a value for a specific criterion, that criterion is excluded for that option only.
- The remaining weights are re-normalized, ensuring:
    - **No artificial penalty** for honest uncertainty.
    - **Mathematical correctness** and consistency.
    - **Deterministic outcomes** that users can audit.
- This approach preserves explainability while avoiding silent bias.
- This behavior is applied per-option rather than globally, ensuring incomplete data does not distort the evaluation of fully specified options.

### Structural Limitation & Correction
Initially, the options and criteria emitted by the Structure Agent were not editable, which contradicted the core philosophy of user control.
- **Correction:** The workflow was restructured to explicitly allow user intervention at every step:
    1. Multiline natural language user input.
    2. AI Structure Agent (draft output).
    3. **Confirm / Edit Options.**
    4. **Confirm / Edit Criteria.**
    5. **Confirm Cost vs. Benefit Classification.**
    6. **Confirm or Define Ordinal Scales.**
    7. Rank criteria by importance.
    8. Attribute mapping and engine evaluation.
- This ensured AI output remains **advisory, not authoritative**.

## Phase 8: Input Flow Corrections & UX Guardrails
Extensive CLI testing revealed several UX and flow issues that could lead to incorrect or misleading decisions.

### Key Fixes & Improvements
- **Pipeline Integrity:** Removed `prompt_missing_values` flows that were incompatible with the revised structured pipeline.
- **State Propagation:** Fixed option and criterion editing so changes correctly propagate to attribute mapping, scoring, and explanation.
- **UX Safety:** Added explicit warnings when users delete criteria that may affect ranking integrity.
- **Criterion Definition:** Users must now explicitly choose "Benefit" or "Cost" and define units at the point of creation, preventing downstream semantic errors.

## Phase 9: Structure Agent Corrections & Human Validation
Relying solely on the LLM to infer "Cost vs. Benefit" semantics from natural language proved unreliable in edge cases.

### Problem:
- The Structure Agent sometimes misclassified criteria (e.g., "Distance" could be a cost for a commuter but a benefit for someone wanting a scenic drive).
- Generated JSON lacked consistent semantic guarantees, leading to normalization errors.

### Resolution:
- **Human-in-the-Loop:** Classification was moved to an explicit user confirmation step.
- The Structure Agent now only **suggests** a structure; the user **validates** the semantic meaning before scoring.

## Phase 10: UX Dead Ends, Scale Editing & Explanation Consistency
This phase addressed subtle logic failures that, while technically "correct," confused users or broke the explainability chain.

### Ordinal Scale Refinement:
- **The Issue**: Editing a proposed scale forced users into AI-generated paths, and ignored scales provided no warning about ranking impact.
- **The Fix:** Users now have full control over scale definitions. Explicit warnings are shown for ignored criteria, and edited scales are re-mapped deterministically via the Attribute Mapper.

### Ranking & Validation:
- **Unique Ranking:** Prevented duplicate ranks to ensure a clear priority hierarchy.
- **Visual Feedback:** Displayed the full criteria list during ranking to reduce cognitive load.
- **Numeric Fallbacks:** Added validation for measurable criteria lacking numeric input, offering the choice to define an ordinal scale, auto-generate one, or ignore the criterion.

### Explanation Pipeline Alignment:
- **The Issue**: Mismatches between normalized internal keys (snake_case) and user-facing labels caused "missing data" flags in the Explanation Builder.
- **The Fix:** Synchronized normalization across the engine, builder, and agent. The Explanation Agent now operates strictly on a pre-validated, auditable payload, ensuring the narrative matches the math perfectly.

## Current System State
The project has evolved into a robust, transparent **Decision Companion System** with the following characteristics:
- **Stable Pipeline:** A reliable end-to-end flow from messy text to narrated results.
- **Mathematical Rigor:** Deterministic rankings with proper handling of cost/benefit and missing data.
- **User Agency:** AI is used strictly for assistance, with humans retaining final authority over all model parameters.
- **Grounded Explanations:** Narratives that are mathematically honest and auditable.

>Overall, the system evolved from a static scoring prototype into a flexible, user-controlled decision framework that remains deterministic, explainable, and resilient to real-world ambiguity.

## Future Roadmap

With the core architecture stabilized, future development will focus on expanding the system's depth and accessibility:

### 1. Decision Persistence & Reusability
Implement a lightweight persistence layer (e.g., SQLite) to allow users to save, name, and revisit past decisions, enabling long-term tracking of evolving priorities.

### 2. Sensitivity & "What-If" Analysis
Add tools to observe how adjusting a criterion's weight impacts rankings in real-time, helping users understand their confidence in the final result.

### 3. Hard Constraints & Filtering
Introduce optional "hard constraints" (e.g., maximum budget or minimum safety rating) to automatically filter non-viable options before the scoring engine begins.

### 4. Enhanced Interface & Visualization
Develop a web-based UI (e.g., React or Streamlit) to visualize score breakdowns and weight distributions through interactive charts (radar plots, stacked bar charts).

### 5. Comprehensive Testing Suite
Transition from inline assertions to a full `pytest` suite with mocked AI responses to ensure 100% coverage of mathematical and logical edge cases.
