# Task: Pre-Implementation Audit of the Product Blueprint

Act as a senior full-stack engineer, software architect, product engineer, UX reviewer, security specialist, and technical project planner.

Your task is to conduct a thorough, critical audit of this repository before implementation begins.

The repository contains the product blueprint for a Persian, RTL-first social platform dedicated to the Iranian stock market. It includes Markdown documentation covering product requirements, features, UX/UI, architecture, data models, development plans, and related topics.

**Your goal is NOT to revise the documentation or start building the application. Your goal is to determine what is wrong, missing, ambiguous, inconsistent, impractical, or insufficiently specified, then ask me the questions needed to resolve these issues.**

I will provide the answers and make the product decisions. Do not make those decisions for me.

## 1. Inspect the Entire Repository

Read all relevant Markdown files and project instructions, including the README, specifications, architectural documents, feature descriptions, development plans, and any existing implementation-related files.

Follow cross-references between documents. Do not audit files in isolation.

Build a clear understanding of the intended product, its users, its features, its business rules, its proposed architecture, and its implementation plan before reporting your findings.

If a file references another document that is missing, identify it. If the repository is too large to inspect in one pass, work systematically through it and keep track of what has and has not been reviewed.

## 2. Conduct a Critical Technical Audit

Review the blueprint as if you were responsible for building, deploying, securing, and maintaining the finished product.

Look for:

* Contradictions between documents.
* Requirements that are incomplete, vague, or open to multiple interpretations.
* Missing functional requirements and essential social-platform features.
* Missing user journeys, screens, states, permissions, or administrative workflows.
* Unclear distinctions between normal users, verified analysts, institutions, administrators, and public visitors.
* Inconsistent terminology, entities, identifiers, relationships, or business rules.
* Missing database entities, relationships, constraints, or data lifecycle rules.
* API requirements that are undefined or incompatible with other specifications.
* Incomplete handling of stock symbols, hashtags, stock associations, and external market-data APIs.
* Missing authentication, authorization, account recovery, privacy, and security requirements.
* Insufficient moderation, reporting, blocking, spam prevention, and abuse-handling rules.
* Incomplete notifications, feeds, search, pagination, sorting, and content discovery requirements.
* Missing loading, empty, error, offline, permission-denied, and other edge states.
* Incomplete Persian/RTL, responsive design, accessibility, and mobile UX requirements.
* Unclear component boundaries, excessive coupling, duplicated logic, or unnecessary architectural complexity.
* Missing performance, reliability, backup, deployment, monitoring, logging, and testing requirements.
* Features that depend on decisions or infrastructure that have not been defined.
* Development tasks that cannot be implemented reliably because their acceptance criteria are missing.
* Unrealistic delivery plans, hidden dependencies, or work scheduled in the wrong order.
* Any other risks that could lead to rework, security vulnerabilities, poor UX, implementation failures, or maintenance difficulties.

Do not assume the blueprint is correct simply because it is detailed. Challenge its assumptions and assess whether its requirements can actually be implemented.

Do not recommend complexity just to make the architecture appear sophisticated. Evaluate every issue in the context of the product's expected scale and actual needs.

## 3. Identify Missing Features Without Expanding Scope Blindly

Assess whether the blueprint adequately covers the complete experience for:

* Public, unauthenticated visitors.
* Registered users and investors.
* Verified individual analysts.
* Institutional and publisher accounts.
* Organization owners and any additional organization members, if supported.
* Administrators and moderators.
* Users managing their profiles, privacy, preferences, sessions, notifications, and security settings.

Check the complete lifecycle of important features: creation, editing, viewing, interaction, deletion, reporting, moderation, permissions, and relevant edge cases.

Distinguish genuinely necessary features from optional enhancements. Do not add features merely because other social platforms have them.

## 4. Find Conflicts and Missing Decisions

For every significant finding, identify the specific document and section involved.

Explain:

1. What the current documentation says or fails to specify.
2. Why this is a problem.
3. What could go wrong during implementation if it remains unresolved.
4. What clarification or decision is required from me.

When documents contradict one another, identify both sides without choosing a winner.

When information is missing, explain the available interpretations where useful, but do not silently select one.

When you identify a potentially better solution, you may mention it as an option. Do not implement it, update the specification, or treat it as approved.

## 5. Ask Me Questions — Do Not Answer Them Yourself

This is the most important instruction.

Your output must focus on **questions that require my clarification or decisions**.

Examples include:

* What exactly should happen when a post contains multiple stock hashtags?
* Should every registered user be allowed to publish posts, or should publishing require a particular account status?
* Who can verify an analyst or institution, and what evidence is required?
* Can institutional accounts have multiple members, and what can each member do?
* What should happen to posts and comments when an account is suspended or deleted?
* Which market-data API is authoritative, and what should users see when it is unavailable?
* Which features are mandatory for the first release?
* Which permissions and administrative actions must be available?

These are examples, not assumptions about the correct answers.

Ask questions only when the answers materially affect product behavior, architecture, security, UX, scope, or implementation.

Do not ask questions whose answers are already explicitly and consistently documented. If a requirement is documented but appears technically flawed, explain the flaw and ask whether the intended behavior should remain unchanged or be reconsidered.

Group related questions together. Prioritize blocking decisions before minor preferences. Where several independent questions exist, ask them in manageable batches rather than overwhelming me with an unstructured list.

Do not ask me to make low-level technical decisions unnecessarily. For implementation details that can safely be decided by an experienced developer without changing the intended product, identify them as implementation choices rather than blocking questions.

## 6. Required Audit Report

Create your audit report in the chat response. You may also create a separate audit report file if useful, but do not modify any existing project documents.

Organize your findings into these sections:

### A. Overall Assessment

Summarize the blueprint's readiness for implementation, its strongest areas, and its biggest risks. Do not give a readiness verdict without supporting findings.

### B. Critical Blockers

Issues that must be clarified before implementation can safely or coherently begin.

### C. Important Gaps and Flaws

Missing requirements, inconsistencies, architectural concerns, UX problems, security issues, and other significant risks.

### D. Missing or Underspecified Features

Identify missing functionality by user type, product area, and administrative responsibility.

### E. Documentation Conflicts

List conflicting requirements and the exact documents or sections involved.

### F. Implementation Risks

Identify tasks that cannot be implemented reliably, dependencies that are unresolved, and acceptance criteria that are insufficient.

### G. Questions for Me

Provide a numbered, prioritized list of clear questions. For each question, explain briefly why the answer matters and reference the relevant document or section.

Where appropriate, include the plausible options to help me answer, without presenting any option as the default or silently making the decision.

### H. Non-Blocking Technical Decisions

Identify implementation details that do not require my product-level input but should be resolved by the developer before or during implementation.

### I. Audit Coverage

List the documents reviewed, any documents you could not access, and any parts of the audit that remain incomplete.

## 7. Repository and Workflow Restrictions

* Do not write application code.
* Do not implement features.
* Do not revise, rewrite, reorganize, or delete existing documentation.
* Do not change the architecture or create a replacement development plan.
* Do not resolve conflicts by silently editing files.
* Do not create commits or push changes.
* Do not claim to have reviewed files you have not actually inspected.
* Do not mark assumptions as confirmed requirements.
* Do not start implementation until I have answered the necessary questions and explicitly authorized the next phase.

You may inspect the repository, trace references, compare documents, and perform read-only analysis.

## 8. Continue Collaboratively

Start by completing as much of the audit as possible from the existing documentation.

Then present your findings and ask me the highest-priority questions.

After I answer, assess which issues are resolved and which remain open. Ask focused follow-up questions where necessary.

Maintain an explicit distinction between confirmed requirements, unresolved questions, proposed options, and implementation decisions.

**Stop at the audit and clarification stage. Your deliverable is a precise account of what needs attention and what you need to know from me. I will provide the missing information before we create a revised, implementation-ready plan.**
