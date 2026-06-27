---
name: repo-analyzer
description: Use when the user mentions "analyzing a project", "analyzing a repository", "analyzing GitHub", "project analysis", "source code analysis", "architecture analysis", "code analysis", "studying this project", "studying this framework", "seeing how this library is implemented", "comparing two projects", "project evaluation", "framework evaluation"
---

# In-depth Git project analysis skills

We provide in-depth analysis of open-source projects and generate professional architecture reports. These reports offer profound technical insights, enabling readers to understand business problems, master architectural design, and develop their own critical thinking.

## When to Use

- Analyze the architecture and design of open source projects
- Compare the design differences between two similar projects
- In-depth study of the implementation ideas of a framework or library

## When NOT to Use

- Simple code issues or debugging
- Single file analysis or code review
- No code modifications required at the architectural level

## Output Language

The default language is Chinese. If the user asks a question in another language, the language used will be the same.

## Core Principles

### 1. Prioritize Business Perspective

Start with "What problem does this project solve?", not "What functions are in this file?"

| Don't | Yes |
| ----- | --- |

The `handleRequest(ctx)` function accepts a Context parameter... After a request comes in, the system will go through three stages: authentication, rate limiting, and route distribution...
| `interface MessageQueue { push(); pop() }` | Modules are decoupled through message queues; producers only need to post messages, and consumers pull messages according to priority.

### 2. Controlling the level of abstraction: Talking about design, not code.

By default, descriptions are presented at the design pattern and architecture level; **raw code is not shown unless absolutely necessary**. The focus is on the process, logic, and design principles, expressed using architecture diagrams (Mermaid), flowcharts, and tables, rather than code snippets. Code is only shown when the design is exceptionally ingenious, the project has a unique original concept, or the implementation is a core selling point, and in such cases, it must first be explained in natural language.

### 3. Global Associations

Every local analysis must be connected to the overall project design philosophy—this is the key difference between "code specifications" and "architectural analysis." See the global connections section in [analysis-guide.md](references/analysis-guide.md) for details.

### 4. Inspirational Writing

The goal is to **learn something and stimulate thinking**, not just provide a code manual. It's like a senior engineer explaining something on a whiteboard—with viewpoints, reasoning, and comparisons. See the inspirational writing section of [analysis-guide.md](references/analysis-guide.md) for details.

### 5. In-depth Insight: Why > What (Mandatory)

Every design decision must be explained in terms of motivation, trade-offs, and the costs of alternatives. Describing "what" is just the starting point; explaining "why" is where the value of the analysis lies. Every core module and the overall architecture must answer:
Why design it this way? It's not just about "what pattern was used," but rather "why it's suitable for this scenario."
What would happen if we didn't do it this way? The cost of the alternative.

- **Gap with industry best practices?** Areas of leadership and room for improvement.
- **If you were to redesign it?** Demonstrate a deeper understanding.
- **A systematic design philosophy?** A style that runs throughout the project (e.g., "convention over configuration," "zero-cost abstraction")

Example:

> ❌ The routing system uses a middleware pattern and supports chained calls.
>
> ✅ The routing system opted for an onion model instead of a linear pipeline. While linear pipelines are simpler to implement, the onion model allows each middleware to handle both request and response phases simultaneously—crucial for logging, timing, and error recovery. Express initially chose a linear model but later had to resort to various hacks to handle post-response logic; Koa learned from this experience and switched to the onion model. If I were to redesign it, I would consider adding middleware dependency declarations so the framework can automatically sort them—this is what Fastify does, avoiding hidden bugs caused by order.

### Supplementary Requirements

- **Code as the Standard** — All conclusions are supported by code, clearly marked with `file path` or `file path:line number range`. Vague descriptions are prohibited.
- **Human Touch** — Like a senior engineer giving an onboarding session to a new colleague, incorporate subjective evaluations and suggestions, avoiding AI-sounding clichés.
- **Focus on Key Points, Briefly Cover Secondary Aspects** — In-depth analysis of core innovations, with general utility functions briefly mentioned.
- **Critical Thinking** — Compare with industry practices, point out real problems, and do not shy away from shortcomings. See [analysis-guide.md](references/analysis-guide.md)
- **Fluent and Easy-to-Understand Writing** — The overall writing style should be fluent and natural, allowing even novice engineers to understand and learn from it. Avoid overly academic language or piling up jargon.
- **Avoid a rambling, disjointed account** — Each module should demonstrate in-depth detail, avoiding brief mentions or superficial discussions. If appropriate, each module should be accompanied by a corresponding Mermaid architecture diagram, ensuring readers are inspired and grasp the essence of the design.

## Analyzing Workflow

**Flexibility Principle:** All stages and sections below are advisory guidelines, not a checklist that must be strictly followed. The agent should make dynamic decisions based on the characteristics of the project being analyzed—if a stage or step is not relevant to the current project, it can be skipped or simplified. The quality of the final report is paramount.

### Phase 1: Project Acquisition and Initialization

1. Parse user input (supports `owner/repo`, GitHub/GitLab/Gitee URL, local path, project name)
2. Create a workspace: Create a directory `repo-analyses/${REPO_NAME}-{YYYYMMDD}` in the user's home directory as `$WORK_DIR` (cross-platform: macOS/Linux use `$HOME`, Windows use `$USERPROFILE` or `$HOME`).
3. If the user provides a local path, skip the clone step; otherwise, clone the repository using `git clone --depth=1`.
4. Obtain basic metadata (Star, Fork, Contributors, Code Statistics)

### Phase 2: Project Scale Assessment and Analysis Model Selection

1. **Count the number of valid lines of code** (excluding skippable code), and list the distribution by module.
   - Code definitions can be skipped: test code, build/deployment configuration (Dockerfile, CI yaml, etc.), automatically generated code (protobuf generation, lock files, etc.), and sample/documentation code.
   - Use tools like `find` + `wc -l` or `cloc` to perform statistics, grouping by top-level directory.
2. **Report code size to users**, using AskUserQuestion to allow users to select analytics modes:

| Pattern                         | Core Module Coverage | Secondary Module Coverage | Applicable Scenarios                           |
| ------------------------------- | -------------------- | ------------------------- | ---------------------------------------------- |
| Quick Analysis                  | ≥30%                 | ≥10%                      | Quickly understand the overall project picture |
| Standard Analysis (Recommended) | ≥60%                 | ≥30%                      | Conventional Architecture Analysis             |
| In-depth analysis               | ≥90%                 | ≥60%                      | Thoroughly examine each design decision        |

3. Write the code size statistics and user-selected analysis mode into `drafts/03-plan.md`, and use this to control the analysis depth in subsequent stages.

**Coverage Calculation Rules**:

- Coverage = Union of line ranges actually requested by the Read tool / Total number of lines in the file
- For large files (>500 lines), they must be read in segments to ensure that the following critical sections are covered:
  - Type definitions and imports in the file header (first 100 lines)
  - Core business logic functions (located by directory structure or function name)
  - Test code at the end of the file (if any)
- Reading only a small portion of a file (<30%) is not counted in the coverage rate and is considered "unread".
- Automatic code generation (proto files, lock files, etc.) can reduce coverage requirements: scanning the structure is sufficient, eliminating the need for line-by-line reading.

### Phase 3: External Research + Project Document Study (Search First, Then Read)

1. Use WebSearch to search for project evaluations, comparisons, and architecture discussions (at least 3-5 searches).
2. **Traverse the project's official website** (if it exists):
   - Extract the official website URL from the README or GitHub page
   - Use WebFetch/tavily_crawl to traverse key pages on the official website (Homepage, Features, Use Cases, Comparison, Blog, etc.).
   - Key takeaways: Product positioning tagline, typical use cases, official competitor comparison, user case studies/testimonials
     Official website content is often the best source for understanding "why this product is needed," as it's more direct than code and technical documentation.
3. **Read through the project's built-in documentation:**
   - Architecture documentation (directories such as `architecture/`, `docs/`, `design/`, etc.)
   - Developer guidelines such as CONTRIBUTING.md and AGENTS.md
   - RFC, ADR (Architecture Decision Records), design proposals, etc.
     These documents often contain the developer's design thinking, trade-offs, and historical decision-making background, making them invaluable primary sources for understanding "why it was designed this way."
   - Extract key design decisions and ideas from the document into your research notes.
4. Compile research findings and write them into `drafts/03-research.md`. This must include the following structured paragraphs (mark "Not Found" if insufficient information is found):
   - **Core problem addressed by the project:** Describe the pain point using 1-3 specific scenarios (who, under what circumstances, what problem encountered, and why existing solutions are insufficient).
   - **Competitor/Similar Project Comparison**: List 3-5 of the most relevant competitors, explaining their differences in positioning and technical approaches.
     Why is this project necessary? Can't we solve it using existing solutions? What is the unique value proposition of this project?
   - **Organizational motivations behind the project** (if applicable): Strategic considerations of commercial companies, ecological positioning of the open-source community.
5. Generate the analysis plan and write it to `drafts/03-plan.md`

Phase 4: Project Feature Recognition + Adaptive Questioning

This is the core phase. Instead of using a fixed list of questions, we generate targeted questions based on the characteristics of the project.

**step:**

1. **Quick Scan:** Scans the entry file, directory structure, dependency declarations, project documentation, and README file.
2. **Identify the core features of the project:**
   - Project type and positioning (library/framework/application/tool)
   - Scale and maturity
   - Design style signals (typography, minimalist API, configuration-driven, etc.)
   - Technology stack characteristics (emerging technologies, multiple languages, specific runtime)
   - Community positioning (core infrastructure, application-layer tools, educational projects, etc.)
3. **Extracting Questions from Features:** Generate targeted questions based on observed project characteristics. Questions should help focus the analysis, not simply follow a process.

   **Thinking Process**—Every observation may suggest a question worth asking the user:
   - Observed technology choices → Ask about the motivation (uncommon technology combinations? Implementing functionality that is usually solved using third-party libraries?)
   - Observed architectural features → Ask for priorities (performance optimization traces? complex plugin/extension systems?)
   - Observed design tensions → Trade-offs (simplicity vs. flexibility? The burden of backward compatibility?)
   - Observed project positioning → Ask the audience (Who are the target users? Is it a replacement or a gap filler in the ecosystem?)

   **Dimensional Inspiration**—What project characteristics suggest what analytical perspectives:
   - Small, concise libraries → API design philosophy, boundary definition; Large frameworks → Modular strategy, backward compatibility, ecosystem governance
   - Using emerging technologies → Why choose it, migration costs; Multilingual/Multi-paradigm → Language boundary design
   - Extensive generics/type manipulation → Type safety vs. complexity tradeoffs; Minimalist APIs → How is simplicity achieved, and what is sacrificed?

   **Characteristics of good questions:** Specific (based on phenomena observed in the code), analytically valuable (the answer influences the direction of analysis), answerable by users (asks about concerns and preferences, not technical details requiring in-depth code analysis), and non-repetitive (avoids questions that can be answered simply by looking at the code).

4. **Asking questions to users:** Use the AskUserQuestion tool to ask users questions, no more than 3 questions at a time.
   One issue to clarify is the level of detail at the beginning of the report: For well-known projects, users may not need lengthy product introductions and competitor comparisons, and may simply want to jump straight to code analysis. Ask users if they need a contextual introduction and competitor positioning section, or if they prefer to start directly with a project overview and code analysis.
5. **Unlimited rounds:** Multiple rounds of questioning are allowed until the direction is clear. Further questions can be asked if new key points of disagreement are discovered during the analysis.

**Key Principle:** The problems should be entirely driven by the characteristics of the project, without pre-defined categories. Different projects should generate completely different problems.

### Phase 5: Dynamic Report Structure Design

The chapter structure of this report was designed based on user responses and project characteristics.

**step:**

1. **Comprehensive Information**: Combining the research from Phase 3, the project characteristics from Phase 4, and user responses.
2. **Design Chapter Structure**: Do not use a fixed template, but the structure must meet the skeleton constraints (see below).
3. **Outline Report:** Provide the user with the designed report outline for confirmation before proceeding.
4. **Identification Module**: Tracks the core data flow and identifies N logical modules (divided according to business functions), categorized into core modules and secondary modules.
5. **Design Module Narrative Thread**: Determine the presentation order and transition logic of modules within the report, organizing them not according to a table of contents structure, but rather according to the best path for reader comprehension.
   - Choose a narrative thread: data flow driven (which modules a request passes through from entry to exit), layered driven (from bottom to top), or problem driven (expanding layer by layer from the core problem to the solution).
   - Specify the transition logic between every two adjacent modules: the output/problem/limitation of the previous module → the necessity of introducing the next module.
   - Write the narrative thread into `drafts/05-modules-plan.md`, with an example format: Module A → [The output of A needs to be consumed by B] → Module B → [B solves X but introduces problem Y] → Module C
6. **Write Plan**: Output module list and report outline are written to `drafts/05-modules-plan.md`

**Skeleton Constraints** (The report does not specify a particular section, but these constraints must be met):

- Introduce a **scenario-based question** (using specific scenarios to clearly explain what problem the project solves, the shortcomings of existing solutions, and why this project is needed—materials from Phase 3 research notes). **Note**: If the user indicates in Phase 4 that they do not need a lengthy introduction (e.g., the project is already well-known), this section can be simplified or skipped, and you can start directly from the project overview.
- **Competitive positioning** (the key differences from similar projects are not a comparison of feature lists, but rather differences in design philosophy and technical approach). **Note:** As above, users can choose to skip this.
- Includes a **project overview** (allowing readers to quickly understand what the project is and what problem it solves).
- Includes in-depth analysis (the reasons behind the core design, trade-offs, and comparisons with industry practices).
- Includes **evaluation and insights** (the advantages and disadvantages of honesty, and what readers can learn from it).
- Includes **architecture visualization** (Mermaid charts)
- All conclusions are supported by code.

### Phase 6: Parallel Deep Analysis (Subagent Team)

The subagent must be started in parallel using the Agent tool. Refer to the prompt template and collaboration guidelines in [module-analysis-guide.md](references/module-analysis-guide.md).

Each subagent's prompt must include the overall project design philosophy and global perspective requirements to ensure that module analysis is not isolated.

Each subagent's prompt must also include the narrative context of that module (from the narrative design in Stage 5): what the previous module covered, what questions the reader enters with in this module, and what this module needs to set the stage for the next module. The subagent should begin its draft with 1-2 sentences connecting to the previous module and end with 1 sentence setting the stage for the next module.

Each subagent's prompt must end with a coverage requirement (refer to the coverage requirement section in module-analysis-guide.md), indicating the current analysis mode and minimum coverage target. A detailed coverage table must be attached to the end of the draft.

**Subagent Write Strategy**:
For large modules (total number of lines > 5000), incremental writing of drafts must be requested in the subagent prompt:

- After completing the analysis of each subsystem/submodule, immediately write that section into a draft file.
  The first subsystem uses Write to create files, and subsequent subsystems use Edit to append.
- Do not wait until the entire file has been read before writing it all at once.
- Coverage details will be added at the end.

**Main agent waiting discipline**:

- After the subagent starts, the main agent must not read the source code files that the subagent is responsible for.
- During the waiting period, the main agent should focus on: reading project documents (architecture/, docs/), conducting external research, designing the report skeleton, and preparing the integration framework for Phase 8.
- The criterion for determining if a subagent is stuck is: no new lines have been added to the output file for more than 5 minutes. Only after confirming a stuck state can the main agent take over the analysis of that module. \***\*Premature merging is strictly prohibited:** The merging process for Phases 7 and 8 must only begin after all subagents have completed. Do not start writing the final report while some subagents are still running.

### Phase 7: Cross-validation + Quality Control (Main Agent)

**7.1 Coverage Gating**:

1. Read the coverage details table at the end of each `drafts/06-module-*.md` file.
2. Quick check: Does each draft end with a coverage table, and are the totals marked as meeting the standards? (✅/❌)
3. Only modules marked with ❌ or lacking a coverage table require in-depth inspection.
4. Non-compliant modules → The main agent automatically supplements the reading of key documents that were not covered, and adds the supplementary findings to the corresponding draft.
5. If the requirements are still not met after supplementation → Report to the user which modules are not meeting the requirements and the reasons (e.g., files are too large, binary files, etc.).

**7.2 Spot Check Verification**:

1. Select 2-3 key conclusions from the draft of each core module.
2. Go back to the source code and verify the accuracy of the conclusion line by line.
3. If deviations are found, revise the corresponding content in the draft.

**7.3 Cross-validation**:

1. Cross-validation of cross-module conclusions labeled "Pending Main Agent Validation"
2. Provide comprehensive answers to the exploratory questions and identify cross-module design patterns.
3. Verify global coherence: Are the analyses of each module connected to the overall project design philosophy?
4. Write to `drafts/07-cross-validation.md`

### Phase 8: Multi-source fusion and final report (primary agent)

1. Extract architectural insights and a systematic design philosophy
2. Deepen competitor comparison based on Phase 3 research results (supplementary searches only when Phase 3 information is insufficient).
3. Propose improvements "if redesigned".
4. Write to `drafts/08-insights.md`
5. **Multi-Source Fusion:** Using the report chapter structure designed in Phase 5 as the framework, content is extracted from various drafts to fill in the gaps. When the same concept appears in multiple drafts, the most detailed version is used, supplemented with information unique to other versions. After fusion, all "See Draft X" or "See Appendix" navigation indicators are removed.
   - **Narrative Coherence**: Organize module chapters according to the narrative line designed in Phase 5. Each module chapter must begin with 1-2 transition sentences connecting to the conclusion or question of the previous module. Avoid abrupt transitions such as "Next, we will analyze module X," and instead use natural transitions (e.g., "The Gateway completes the authentication and routing of requests, but it is only responsible for 'who can come in,' not 'what can be done after coming in.' This responsibility for behavior control is undertaken by the strategy engine.").
6. **Segmented Writing**: The final report usually exceeds 500 lines. First, write the first few sections (200-300 lines), and then append the rest using Edit. Before each append, use Read to confirm the end position.
7. **Coverage Summary**: Summarize the coverage data and write it to `drafts/08-coverage.md` (do not include it in the final report).
   - Data is extracted directly from the coverage details table at the end of each subagent's draft, without requiring the main agent to recalculate.
   - If the main agent adds additional reads in phase 7, the added lines will be added to the "Read Lines" count of the corresponding module.
     Summary table format:

| Module | Type           | Number of Files | Valid Lines of Code | Number of Lines Read | Coverage | Meets Standard |
| ------ | -------------- | --------------- | ------------------- | -------------------- | -------- | -------------- |
| ...    | Core/Secondary | ...             | ...                 | ...                  | ...%     | ✅/❌          |

8. Compile and generate the final report (excluding the coverage section).

### List of Draft Files

All intermediate processes are saved to `$WORK_DIR/drafts/`:

| Stage | Document                                    |
| ----- | ------------------------------------------- |
| 3     | `03-research.md`, `03-plan.md`              |
| 5     | `05-modules-plan.md`                        |
| 6     | `06-module-{name}.md` (subagent generation) |
| 7     | `07-cross-validation.md`                    |
| 8     | `08-insights.md`, `08-coverage.md`          |

Files are written in blocks, with each block not exceeding 300 lines or 15KB.

## Special Scenarios

- **Very large projects (>50,000 lines):** Prioritize analysis of core modules, using agents for parallel analysis.
- **Comparative Analysis Mode**: Two projects complete phases 1-4 respectively, and then in phase 5, design a comparative report structure, adding "Design Decision Comparison" and "Selection Recommendations" to the skeleton constraints.

## Output Requirements

1. The final report is a single Markdown file: `$WORK_DIR/ANALYSIS_REPORT.md`
2. Extensive use of Mermaid charts to display architecture, processes, and data flows.
3. For developers who need to understand the business architecture.
4. A framework for evaluating strengths and weaknesses [analysis-guide.md](references/analysis-guide.md)
5. Analytical Philosophy and Depth Standards Reference [analysis-guide.md](references/analysis-guide.md)
