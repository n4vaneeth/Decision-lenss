# Decision-lenss
# Decision-Lens

## Understanding of the Problem
The goal of this project is to build a domain-agnostic Decision Companion System that helps users reason through complex, real-world decisions in a structured and transparent way.

Many decisions—whether about careers, purchases, travel, or personal trade-offs—involve multiple options, competing criteria, incomplete information, and subjective preferences. Users often struggle not because they lack intelligence, but because the decision space itself is unstructured and the trade-offs are unclear. This system is designed to organize that chaos, not to replace human judgment.

Instead of making decisions on behalf of the user, the system assists the user by:

- Structuring messy, natural-language input into explicit options and criteria
- Making trade-offs visible through weighted evaluation
- Producing a ranked outcome along with a clear explanation of why that outcome occurred

A key requirement of the problem is trust. For a decision aid to be useful, users must be able to understand how and why a result was produced. For this reason, the system deliberately avoids using AI to score, rank, or guess outcomes. All scoring and ranking logic is deterministic, auditable, and repeatable.

AI is used only where it adds value without compromising trust—such as helping structure user input or converting deterministic results into human-readable explanations. Its role is strictly constrained, and it is never allowed to invent criteria, modify weights, or influence the final decision logic.
- In short, the problem is not “choosing the best option,” but helping users make sense of their own priorities and constraints in a clear, explainable way across different decision domains. This project addresses that problem by combining structured decision logic with carefully limited AI assistance.

## Table of Contents

| Section | Navigation |
| :--- | :--- |
| **Project Overview** | [Understanding of the Problem](#understanding-of-the-problem) |
| **Core Assumptions** | [Assumptions Made](#assumptions-made) |
| **Solution Design** | [Why the Solution Is Structured This Way](#why-the-solution-is-structured-this-way) |
| **Architectural Decisions** | [Design Decisions and Trade-offs](#design-decisions-and-trade-offs) |
| **Robustness & Validation** | [Edge Cases Considered](#edge-cases-considered) |
| **Installation & Setup** | [How to Run the Project](#how-to-run-the-project) |
| **System Architecture** | [Project Architecture](#project-architecture) |
| **Data Flow** | [Data Flow Diagram](#data-flow-diagram) |
| **Future Roadmap** | [What I Would Improve With More Time](#what-i-would-improve-with-more-time) |

## Assumptions Made

- Users may approach the system with varying levels of clarity. Some users already know their options and criteria, while others may have only a vague idea of what they want.

- The system assumes that not all information will be available upfront. Users may skip values, be unsure about exact numbers, or prefer qualitative descriptions (e.g., “high”, “low”, “decent”) instead of precise data.
Ordinal (qualitative) judgments can be meaningfully converted into numeric representations as long as the relative ordering is explicitly defined by the user.

- When users are unsure how to structure their decision, AI assistance may be used optionally to help extract options and criteria from free-form input. This assistance is advisory and never authoritative.

- Users always retain full control over the decision model. They can add, remove, edit, or override options, criteria, scales, and values at any stage of the process.

- All final rankings are generated using deterministic, transparent logic. AI is never used to assign weights, score options, or influence the final ranking outcome.

- For benefit-type criteria, higher normalized values always represent better performance.

- For cost-type criteria, lower real-world values are transformed such that lower cost corresponds to higher utility during normalization.

- The system assumes that relative comparisons and trade-offs are more valuable to users than absolute precision. As a result, the model prioritizes consistency and explainability over perfect real-world accuracy.

## Why the Solution Is Structured This Way

| Problem / Constraint                           | Design Choice/ Solution                                                                                                   |
| ---------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| Users think in messy, unstructured ways   | **Free-form natural language input** is accepted and then progressively structured into discrete options and criteria using AI-assisted analysis, mirroring how humans naturally describe problems. |
| Users may not know all criteria upfront   | An **iterative, step-by-step confirmation flow** allows criteria to be added, removed, or modified at any stage, acknowledging that clarity often emerges during the decision-making process. |
| Not all criteria are numeric               | The system provides **robust support for both numeric and ordinal (qualitative) criteria**, allowing users to use descriptors like "Excellent" or "Poor" through user-defined or auto-generated mapping scales. |
| Users struggle with trade-offs | A **Weighted Multi-Criteria Decision Model (MCDM)** is used to make importance explicit. By forcing a relative ranking of criteria, the system quantifies trade-offs that are usually handled poorly by human intuition. |
| Black-box AI recommendations reduce trust | A **Deterministic Scoring Engine** ensures that rankings are 100% reproducible and auditable. The system prioritizes mathematical transparency over "black-box" AI-generated rankings. |
| AI can hallucinate or overstep | **Restricted AI Role:** AI is strictly confined to structuring messy input and explaining results. It is architecturally prohibited from assigning weights, scoring options, or influencing the final ranking. |
| Users may not know exact values | **Graceful Handling of Missing Data:** The system uses weight renormalization to handle missing values, providing explicit warnings rather than making silent, potentially incorrect assumptions. |
| Different criteria have different semantics | **Explicit Benefit vs. Cost Classification:** Users confirm whether each criterion is a benefit or a cost before scoring begins. The decision engine applies the corresponding normalization logic accordingly. |
| Explanations often don’t match results | **Output-Driven Explanations:** The explanation payload is derived directly from the scoring engine's raw data, ensuring the natural language narrative is an honest reflection of the underlying math. |
| Users want to “see why” one option won | **Per-Option Factor Breakdown:** Every option is presented with its top positive and negative contributors, highlighting exactly which criteria drove its final rank. |
| Decision logic should work across domains | A **Domain-Agnostic Core** allows the same engine to be used for everything from choosing a car or a job to evaluating complex business software or travel destinations. |
| Criteria have different units (e.g., $ vs km) | **Min-Max Normalization** scales all raw values to a uniform 0.0–1.0 range, preventing criteria with large absolute numbers (like price) from unfairly dominating the final score. |
| Not all criteria are equally important | **Unique Importance Ranking:** Users provide a unique 1-N rank for each criterion, which is then mathematically converted into importance weights that sum to 1.0. |
| Qualitative descriptors are subjective | **User-Defined Ordinal Scales:** Users define the exact sequence of qualitative terms (e.g., "Good > Decent > Bad"), ensuring the system interprets descriptors according to the user's personal context. |

### Underlying Decision Model

The system is structured around a **Weighted Multi-Criteria Decision-Making (MCDM)** approach:

- **Multi-Criteria Evaluation:** Each option is evaluated against multiple, independent criteria.
- **Weighted Importance:** Each criterion is assigned an importance weight by the user.
- **Normalized Scoring:** Each option receives a normalized score (0 to 1) per criterion to ensure fair comparison across different units.
- **Weighted Sum Calculation:** Final scores are computed as a weighted sum, representing the overall utility of each option.

This approach was chosen because it is:
- **Simple and Well-Established:** Grounded in decision science.
- **Transparent and Explainable:** Every part of the score can be audited.
- **Flexible:** Works across any decision domain (cars, jobs, products, etc.).
- **User-Centric:** Easy for users to understand, inspect, and modify.

Most importantly, this structure ensures the system helps users **reason about trade-offs**, rather than hiding logic behind a black-box recommendation.

## Design Decisions and Trade-offs

The development of Decision-Lens involved several key architectural choices to balance user experience, transparency, and technical robustness.

### 1. Deterministic Core vs. AI-Driven Decisions
- **Decision:** The core decision-making logic is strictly deterministic and rule-based. AI is utilized only for auxiliary tasks like structuring messy natural language input and generating human-readable explanations.
- **Rationale:** Prioritizes reproducibility and auditability. Results can be inspected, debugged, and explained step-by-step, preventing the "black-box" hallucinations or inconsistent rankings often associated with LLMs.
- **Trade-off:** The system cannot "intuit" hidden preferences and requires more explicit user input, trading convenience for absolute trust and transparency.

### 2. Weighted Multi-Criteria Decision Model (MCDM)
- **Decision:** Employs a weighted sum model to evaluate options across multiple criteria.
- **Rationale:** Simple, well-established in decision science, and domain-agnostic. It forces users to make their priorities explicit through weights, which helps clarify internal trade-offs.
- **Trade-off:** Assumes linear relationships between criteria and does not capture complex interactions. However, this simplicity ensures the reasoning remains understandable to non-technical users.

### 3. Deterministic Scoring Logic & Normalization

- **Decision:**  
  All options are ranked using a deterministic weighted scoring approach with unit-agnostic normalization.

- **Scoring Logic:**  
  Each option is evaluated against all defined criteria.  
  For every criterion, a normalized score between 0 and 1 is computed for the option.  
  This score is multiplied by the criterion’s normalized weight.  
  The final score of an option is the sum of these weighted normalized scores across all criteria.

- **Normalization Strategy:**  
  To ensure comparability across criteria with different units and scales, min–max normalization is applied:

  - **Benefit Criteria (higher values are better):**  
    Values are scaled such that the lowest observed value maps to 0 and the highest maps to 1, with intermediate values linearly interpolated.

  - **Cost Criteria (lower values are better):**  
    Values are inversely scaled so that the lowest observed value maps to 1 and the highest maps to 0.

- **Rationale:**  
  This approach guarantees mathematical transparency and consistent evaluation across heterogeneous criteria (e.g., price, performance, weight).  
  The scoring process is entirely deterministic; no AI-generated or inferred values are introduced at any stage.

- **Trade-off:**  
  The method is sensitive to extreme values in small datasets. This trade-off was accepted to preserve strict reproducibility, explainability, and auditability of results.

### 4. Support for Both Numeric and Ordinal Criteria
- **Decision:** The system natively supports both numeric data (e.g., price, efficiency) and ordinal/qualitative judgments (e.g., comfort, safety) through user-defined mapping scales.
- **Ordinal Handling:** Qualitative terms are mapped to numeric ranks (e.g., `poor → 1`, `excellent → 4`) before being normalized using the formulas above.
- **Rationale:** Real-world decisions are rarely purely numeric. Supporting qualitative descriptors like "excellent" or "poor" reduces user friction while maintaining mathematical rigor.
- **Trade-off:** Increases internal complexity for normalization and validation, but significantly improves the tool's relevance to human decision-making.

### 5. Incremental, Interactive Input Flow
- **Decision:** Users are guided through a step-by-step confirmation flow (Options -> Criteria -> Types -> Scales -> Importance).
- **Rationale:** Clarity often emerges during the process. This flow reduces cognitive overload and allows users to correct their model without restarting from scratch.
- **Trade-off:** Leads to more code complexity and a longer interaction cycle compared to a single-form input, but ensures a much higher rate of "correct" models.

### 6. Explicit Handling of Missing or Unknown Data
- **Decision:** Missing values are explicitly tracked and surfaced in explanations rather than being silently filled or ignored.
- **Rationale:** Prevents misleading results and makes uncertainty visible. It encourages users to find missing data if it significantly affects the outcome.
- **Trade-off:** Final rankings may feel "incomplete" if data is sparse, but the system prioritizes honesty over a false sense of completeness.

### 7. Explanation Payload Built from Engine Outputs
- **Decision:** Natural language explanations are generated from a deterministic "explanation payload" derived directly from the scoring engine's raw data.
- **Rationale:** Guarantees that the narrative perfectly matches the underlying math. It prevents AI-generated stories from drifting away from the actual facts of the decision.
- **Trade-off:** Explanations may be less "creative" or varied in tone, but they are guaranteed to be accurate and grounded.

### 8. Data Structures and Internal Representation
- **Decision:** Uses simple, explicit Python data structures (`list` and `dict`) to represent the decision state.
- **Rationale:**
    - **Lists** are used for `options` and `criteria` to maintain order and allow easy modification.
    - **Dictionaries** map `weights` and `ratings` to ensure clear, readable relationships between criteria and values.
    - This structure maps directly to the weighted scoring formula, making the code easier to maintain and audit.
- **Trade-off:** Less abstract than a fully object-oriented model, but the simplicity improves readability and reduces the overhead for basic validation.

### Summary
Overall, Decision-Lens intentionally trades **convenience for transparency**, **automation for user control**, and **complex modeling for explainability**. These trade-offs align with the core mission: helping users think more clearly, not replacing their judgment.

## Edge Cases Considered

Decision-Lens is designed to be robust against "messy" real-world data and user input errors. The following edge cases are explicitly handled:

### 1. Mathematical & Normalization Edge Cases
- **Zero Variance Criteria:** If all options have the same value for a specific criterion (e.g., all laptops cost exactly $1,000), the system avoids division-by-zero errors and assigns a neutral normalized score of `1.0` to all.
- **Disparate Units:** The min-max normalization ensures that a criterion with large numbers (like Price in $40,000) does not mathematically "drown out" a criterion with small numbers (like RAM in 16GB).

- **Flexible Cost/Benefit Classification:** The engine handles the mathematical inversion required for cost criteria (where lower is better) while giving users final control. Through an interactive toggle, users confirm whether a criterion like "Price" is a cost or a benefit, ensuring the final score correctly reflects their personal decision logic.

### 2. Missing or Incomplete Data
- **Partial Attribute Data:** If an option is missing a value for a specific criterion, the system does not fail. Instead, it **renormalizes the weights** for that specific option, distributing the importance of the missing criterion across the remaining available criteria.
- **"Unknown" Values:** Input strings explicitly marked as "unknown" (or skipped during CLI entry) are treated as `None` and handled through the weight renormalization logic.
- **Empty Options/Criteria:** If an option has no data across all criteria, it receives a default score of `0.0`. If a criterion has no data across all options, it is excluded from the calculation.

### 3. User Input & Ordinal Scale Validation
- **Ordinal Descriptor Mismatches:** If a qualitative value (e.g., "mostly good") is detected that isn't in the defined scale (e.g., "poor < okay < good"), the system triggers an interactive resolution flow allowing the user to map it, add it to the scale, or ignore it.
- **Scale Confirmation:** To ensure mathematical mapping matches user intent, all ordinal scales are presented visually in "Worst → Best" order for explicit user confirmation before any scoring occurs.
- **Invalid Numeric Input:** All inputs are validated at entry. If a user provides a string where a number is expected, the system provides a clear error message and re-prompts, preventing silent failures or crashes.

### 4. Logic & Consistency Checks
- **Criteria Type Confirmation:** Users are given an explicit chance to confirm if a criterion is a "Benefit" (more is better) or "Cost" (less is better) before scoring begins, preventing inverted results.
- **Scale Verification:** For qualitative criteria, the system visually presents the "Worst → Best" order to the user for confirmation, ensuring that the mathematical mapping (`1 → 4`) aligns with the user's personal intent.
- **Empty Result Resolution:** If the optional AI "Structure Agent" fails to find options or criteria, the system provides a manual fall-back mode for the user to define their decision model from scratch.

## How to Run the Project

Decision-Lens is a Python-based CLI application. Follow these steps to set it up locally:

### 1. Prerequisites
- **Python 3.10+** installed on your system.
- An **OpenAI API Key** (required for AI-assisted input structuring and explanations).

### 2. Clone the Repository
```bash
git clone https://github.com/your-username/decision-lens.git
cd decision-lens
```

### 3. Set Up a Virtual Environment (Recommended)
```bash
# Windows
python -m venv .venv
.venv\Scripts\activate

# macOS/Linux
python3 -m venv .venv
source .venv/bin/activate
```

### 4. Install Dependencies
The project requires several libraries, including `openai` and `python-dotenv`. Install them all at once:
```bash
pip install -r requirements.txt
```

### 5. Configure Environment Variables
Create a file named `.env` in the project root directory and add your OpenAI API key:
```env
OPENAI_API_KEY=your_actual_api_key_here
```

### 6. Test Your API Key (Optional)
Before running the main application, you can verify that your API key is configured correctly:
```bash
python src/agents/test_openai_key.py
```
If successful, you will see a test message (e.g., a programming pickup line) printed in the terminal.

### 7. Run the Application
Start the interactive decision-making process:
```bash
python src/main.py
```

### 8. Use Sample Test Prompts
If you want to test the system without typing a full problem from scratch, you can copy and paste the example scenarios provided in:
- `test_prompt.txt` (Includes detailed scenarios for **Buying a Car** and **Choosing a Remote Job**).

### 9. Interactive Process Flow
1. **Describe Your Problem:** Type a natural language description of your decision (e.g., "I'm choosing between three laptops for gaming and productivity").
2. **Review & Refine:** The AI will extract a draft of **Options** and **Criteria**. You can add, remove, or edit these manually.
3. **Confirm Metadata:** Define which criteria are "Costs" vs. "Benefits" and confirm any qualitative ordinal scales (e.g., `poor < good < excellent`).
4. **Rank Importance:** Provide a unique priority rank for each criterion.
5. **Get Results:** The engine will compute the weighted rankings and the AI will generate a detailed explanation of the results.



## Project Architecture

The project follows a modular structure, separating the core decision logic from the AI-assisted agents and utility functions:

```text
Decision-Lens/
├── src/
│   ├── main.py                 # Application entry point & orchestration
│   ├── agents/
│   │   ├── structure_agent.py   # AI agent for input parsing (GPT-4o-mini)
│   │   ├── explanation_agent.py # AI agent for result narration
│   │   └── test_openai_key.py  # Utility to verify API configuration
│   ├── core/
│   │   ├── decision_engine.py   # Deterministic weighted scoring logic
│   │   ├── attribute_mapper.py  # Qualitative-to-numeric mapping
│   │   └── explanation_builder.py # Data-driven explanation payload logic
│   └── utils/
│       ├── input_helpers.py     # Interactive CLI helpers & validation
│       └── normalization.py     # Key & value normalization utilities
├── requirements.txt             # Project dependencies
├── test_prompt.txt              # Sample decision scenarios
├── RESEARCH_LOG.md              # Research findings and AI prompt history
├── BUILD_PROCESS.md             # Detailed breakdown of the development lifecycle
├── Dataflow.md                  # Technical documentation of data movement
├── GEMINI.md                    # Project context and architectural roadmap, add it in GEMINI CLI to interact.
└── Data_Flow.png                # System data flow visualization
```

## Data Flow Diagram

The following diagram illustrates how data moves from initial natural language input through the AI structuring layer, the deterministic engine, and finally to the narrated explanation:

![Data Flow Diagram](Data_Flow.png)

## What I Would Improve With More Time

While the current system achieves its goal of transparent, deterministic decision support, there are several areas where additional development would meaningfully improve usability and analytical depth:

### 1. Decision Persistence & Reusability
- **The Problem:** Currently, each decision session is ephemeral; once the CLI is closed, the model is lost.
- **The Improvement:** Implement a lightweight persistence layer (e.g., SQLite) to allow users to save, name, and revisit past decisions. This would enable recurring decision-making and long-term tracking of evolving preferences.

### 2. Sensitivity & Trade-off Analysis
- **The Problem:** The system produces a single ranked outcome but doesn't show how "fragile" that ranking is relative to small changes in weights.
- **The Improvement:** Add "What-If" analysis tools to observe how adjusting a criterion's weight impacts the overall rankings in real-time. This helps users understand their confidence in the final result and identify critical trade-offs.

### 3. Hard Constraints & Filtering
- **The Problem:** All criteria currently contribute through weighted scoring, meaning an option with a deal-breaking flaw could still rank high if it excels elsewhere.
- **The Improvement:** Introduce optional "hard constraints" (e.g., a maximum budget of $30k or a minimum 4-star safety rating). This would automatically filter out non-viable options before the scoring engine begins, better reflecting real-world decision behavior.

### 4. Enhanced Interface & Visualization
- **The Problem:** While the CLI is effective for structured input, interpreting a dense list of scores can be cognitively demanding.
- **The Improvement:** Develop a web-based UI (e.g., React or Streamlit) to visualize score breakdowns, weight distributions, and head-to-head comparisons through interactive charts (radar plots, stacked bar charts), making the final rankings more intuitive.
