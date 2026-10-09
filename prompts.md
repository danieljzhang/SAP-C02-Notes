# AWS Certified Solutions Architect - Professional Exam Preparation Prompts

This document contains reusable prompts for creating and reviewing AWS Certified Solutions Architect - Professional (SAP) exam preparation materials. Before generating or updating study content, verify the current exam code, exam guide, domain names, and domain weights against official AWS sources.

## Core Service Documentation Prompts

### 1. Comprehensive Service Notes Creation

```markdown
Please create comprehensive AWS [SERVICE_NAME] notes for the current AWS Certified Solutions Architect - Professional exam. First verify the current exam code and official exam guide, then tailor the notes to the exam's architecture-level expectations.

Include:
1. Service overview, use cases, and problems it solves
2. Core concepts, components, quotas, and service limits relevant to design decisions
3. Architecture patterns and configuration examples (YAML/JSON where useful)
4. Identity, access, encryption, and network security considerations
5. Integration with other AWS services and common enterprise architectures
6. High availability, fault tolerance, backup, and disaster recovery options
7. Performance, scalability, operational complexity, and cost trade-offs
8. Migration and modernization considerations where applicable
9. Monitoring, troubleshooting, and failure modes
10. Eight to ten original, realistic exam-style scenarios with detailed solution reasoning
11. CLI commands or APIs when they clarify implementation
12. Comparisons with similar or competing AWS services
13. Exam tips, decision criteria, and common traps
14. Quick-reference summary
15. Links to current official AWS documentation, whitepapers, and exam resources
16. Key areas to master and any assumptions or version-sensitive details

Format requirements:
- Use clear Markdown headings and concise explanations
- Include practical code examples and readable architecture diagrams
- Explain why the recommended design satisfies the stated requirements
- Compare viable alternatives when the choice involves meaningful trade-offs
- Use original practice scenarios; do not reproduce exam questions or exam dumps
- Do not invent undocumented service behavior, quotas, pricing, or exam content
```

### 2. Detailed Standalone Service Template

```markdown
I am preparing for the current AWS Certified Solutions Architect - Professional exam. Verify the current exam code and official exam guide before writing.

Generate one complete, production-quality Markdown file for AWS [SERVICE_NAME]. Output only the Markdown file content, with no preface or commentary.

Tailor the material to professional-level architecture decisions. Cover:
- Why the service matters to the current exam objectives
- Core concepts and architecture patterns
- Design constraints, quotas, and failure modes
- Security, IAM, encryption, and network considerations
- Reliability, availability, disaster recovery, and data protection
- Performance, scale, operations, and cost trade-offs
- Integration with other AWS services and multi-account environments
- Migration, hybrid, and modernization patterns where relevant
- Troubleshooting and common design pitfalls
- ASCII architecture diagrams
- Original scenario-based questions with explained answers
- Best practices and official AWS reference links

Use correct Markdown headings and fenced code blocks. Clearly label assumptions and details that must be checked against current AWS documentation. Do not include copied exam questions or unsupported claims.
```

## Gap Analysis and Coverage Assessment

### 3. Exam Objective and Topic Gap Analysis

```markdown
Using the current official AWS Certified Solutions Architect - Professional exam guide and the files in [DIRECTORY_OR_REPOSITORY], assess coverage for [SERVICE_OR_TOPIC]. Verify the current exam code, objective names, and weights from official AWS sources.

Identify:
1. Missing exam-relevant concepts and architecture patterns
2. Advanced capabilities and design constraints not covered
3. Integration patterns across accounts, regions, hybrid environments, and AWS services
4. Security, resilience, migration, operations, and cost considerations that need more depth
5. Common failure modes, troubleshooting decisions, and scenario traps
6. Content that is outdated, redundant, unsupported, or outside the exam scope

Then provide:
- A concise gap analysis
- Recommended files and names, following the repository's conventions
- Five to seven key topics for each proposed file
- Priority and rationale based on the current exam guide
- A coverage assessment with its method and limitations; do not present an unsupported percentage as fact
```

### 4. Exam Domain Review

```markdown
@ [DIRECTORY_OR_DOMAIN] Review this material for the current AWS Certified Solutions Architect - Professional exam.

Verify the current official exam guide first. Assess accuracy, completeness, architecture-level depth, consistency, and alignment with the guide. Identify missing or weak objectives, duplicated coverage, outdated guidance, and unsupported claims. Recommend specific additions or corrections. Do not claim a numerical coverage percentage unless you define and apply a transparent scoring method.
```

## Specialized Architecture Prompts

### 5. Multi-Account and Organization Architecture

```markdown
Create comprehensive architecture notes on [MULTI_ACCOUNT_TOPIC] for the current AWS Certified Solutions Architect - Professional exam. Cover:
- AWS Organizations structure, organizational units, and account boundaries
- AWS Control Tower and landing-zone considerations where relevant
- Delegated administration and service control policies
- Cross-account IAM roles, trust policies, and least-privilege access
- Centralized logging, security, networking, and shared services
- Resource sharing and cross-account service integrations
- Governance, guardrails, exceptions, and operational ownership
- Failure modes, migration considerations, and design trade-offs
- Original scenario-based examples, diagrams, and official references
```

### 6. Service Integration Deep Dive

```markdown
Create detailed architecture notes on [SERVICE_NAME] integrations with AWS and third-party services for the current AWS Certified Solutions Architect - Professional exam. Cover:
- Native AWS integrations and when to use them
- Cross-account, cross-region, and hybrid connectivity patterns
- Identity, authorization, encryption, and secrets handling
- Event, data, and network flows, including key limits or failure points
- Resilience, observability, and operational ownership
- Cost and performance trade-offs
- Common integration failures and their diagnosis
- At least three original architecture scenarios with alternatives and rationale
- Current official documentation links
```

### 7. Advanced Architecture Feature Deep Dive

```markdown
Create comprehensive notes on [ADVANCED_FEATURE_OR_PATTERN] for the current AWS Certified Solutions Architect - Professional exam.

Explain:
- The problem this feature or pattern solves and when not to use it
- Architecture, prerequisites, limits, and dependencies
- Security and governance implications
- Availability, recovery, scale, performance, and cost characteristics
- Integration with related AWS services
- Implementation and migration considerations
- Failure modes, troubleshooting, and common exam traps
- Original decision scenarios comparing viable alternatives
- Official AWS references and version-sensitive details
```

## Quick Reference Materials

### 8. Exam Domain Cheatsheet

```markdown
Create a concise, exam-focused cheatsheet for the current AWS Certified Solutions Architect - Professional exam objective [OBJECTIVE_OR_DOMAIN]. Verify the current exam guide first.

Include:
- Key architecture principles and service-selection cues
- Important service capabilities, constraints, and limits to verify
- Resilience, security, migration, operations, and cost reminders
- Common architecture patterns and when they fit
- Similar-service comparisons and decision points
- Scenario clues, common traps, and elimination tips
- A final review checklist

Use compact bullets and tables for scanning. Keep details accurate and link to official sources where useful. Do not reproduce actual exam questions.
```

### 9. Service and Pattern Comparison

```markdown
Compare [SERVICE_OR_PATTERN_A] with [SERVICE_OR_PATTERN_B] for the current AWS Certified Solutions Architect - Professional exam.

Explain:
- Best-fit use cases and disqualifying requirements
- Functional and architectural differences
- Availability, durability, scaling, performance, and recovery behavior
- Security, governance, and integration considerations
- Operational complexity and cost drivers
- Migration or coexistence options
- Scenario cues that indicate which option is a better fit

Include a concise comparison table and original scenarios. Cite current official AWS sources and identify any assumptions.
```

## Troubleshooting and Operations

### 10. Architecture Troubleshooting Playbook

```markdown
Create a systematic troubleshooting playbook for [SERVICE_OR_ARCHITECTURE] at the level expected for the current AWS Certified Solutions Architect - Professional exam.

Cover:
- Symptoms, likely causes, and diagnostic sequence
- Identity, permissions, policy, and encryption failures
- DNS, routing, connectivity, and hybrid-network issues
- Availability, scaling, performance, and quota bottlenecks
- Data replication, backup, and recovery failures
- Logs, metrics, traces, and relevant diagnostic tools
- Safe remediation, prevention, and architecture improvements
- How to distinguish a root cause from a misleading symptom

Use original scenarios and current AWS documentation. Clearly distinguish verified service behavior from hypotheses.
```

## Repository Management

### 11. File Organization and Coverage Review

```markdown
Analyze the [REPOSITORY_OR_DIRECTORY] structure for current AWS Certified Solutions Architect - Professional exam preparation.

Using the current official exam guide:
1. Identify misplaced, missing, duplicated, or overly broad files
2. Recommend a structure organized around the repository's established conventions and exam objectives
3. Identify cross-cutting topics that should be linked rather than duplicated
4. Flag stale exam codes, outdated service guidance, broken internal links, and unsupported claims
5. Recommend focused additions, consolidations, or file moves

For each recommendation, give the proposed path, priority, key content, and reason. Do not move or rewrite files as part of the analysis unless explicitly requested.
```

### 12. Comprehensive Repository Review

```markdown
@ [REPOSITORY_NAME] Conduct a final review of this AWS Certified Solutions Architect - Professional exam preparation repository.

Review objectives:
1. Verify technical accuracy against current official AWS documentation
2. Assess coverage against the current official exam guide
3. Check that material teaches architecture decisions, not just service definitions
4. Find outdated, contradictory, duplicated, or unsupported content
5. Identify gaps that could weaken exam preparation

Review areas:
- Every current exam objective and its relevant services and patterns
- Multi-account, multi-region, hybrid, migration, and modernization designs
- Security, governance, resilience, disaster recovery, operations, and cost optimization
- Service limits, quotas, IAM examples, diagrams, and configuration snippets
- Scenario quality, answer reasoning, and comparison of valid alternatives
- Internal links, consistent terminology, and clear separation of facts from assumptions

Output:
- Findings ordered by severity with file paths and specific corrections
- Objective-level coverage assessment and the method used
- Critical gaps and recommended priorities
- Accuracy concerns requiring verification
- A concise readiness assessment with remaining limitations

Do not claim access to actual exam questions or guarantee exam readiness.
```

## Exam Preparation Materials

### 13. Final Exam Preparation Strategy

```markdown
Create a final preparation and exam-day strategy for the current AWS Certified Solutions Architect - Professional exam. Verify the current exam format and duration from official AWS sources.

Include:
- A time-management approach suited to the verified exam format
- How to parse long scenarios and separate mandatory requirements from preferences
- Techniques for comparing security, resilience, operational effort, and cost
- Methods for eliminating options that violate explicit constraints
- How to handle multi-account, multi-region, hybrid, migration, and recovery scenarios
- Review and pacing checkpoints
- A final review checklist and post-exam reflection template

Avoid claiming that any topic or question is guaranteed to appear.
```

### 14. Original Practice Scenarios

```markdown
Create [NUMBER] original practice scenarios aligned to the current AWS Certified Solutions Architect - Professional exam guide. Verify the guide and its objectives first. Do not reproduce actual or purported exam questions.

Use a balanced selection of architecture challenges, such as:
- Multi-account governance and cross-account access
- Hybrid connectivity and network segmentation
- Multi-region availability and disaster recovery
- Data migration, replication, and modernization
- Secure workload and identity design
- Scalable data processing and application architecture
- Performance, operational visibility, and fault isolation
- Cost optimization under explicit business constraints

For each scenario include:
- A realistic problem statement with business and technical requirements
- Explicit constraints and relevant assumptions
- The key decision points
- The recommended architecture and why it meets the requirements
- Why plausible alternatives are weaker or unsuitable
- Security, reliability, operational, and cost implications
- A readable diagram where useful
- Common traps and a concise takeaway
```

## Content Updates and Study Planning

### 15. Content Update Verification

```markdown
Review [FILE_OR_TOPIC] for the current AWS Certified Solutions Architect - Professional exam and update recommendations as needed.

Check:
- Alignment with the current exam code and official objectives
- Service behavior, feature availability, quotas, and limits against current AWS documentation
- IAM, security, encryption, and network recommendations
- Architecture and integration patterns, including failure and recovery behavior
- Pricing claims and cost comparisons against current pricing sources
- Code examples, diagrams, and official links

Return specific proposed corrections and sources. Mark details that require verification; do not silently guess or rewrite unrelated material.
```

### 16. Study Plan Creation

```markdown
Create a study plan for the current AWS Certified Solutions Architect - Professional exam, starting [START_DATE], with [DAILY_HOURS] hours available per day and an exam date of [EXAM_DATE]. Verify the current exam guide and format first.

Include:
- A weekly and daily schedule tied to official exam objectives
- Prerequisite review and hands-on architecture practice
- Reading, labs, diagrams, and original scenario practice
- Regular knowledge checks and mock-exam review sessions
- A method to track weak objectives and adjust the plan
- Milestones, rest days, and a final review period
- An exam-day preparation checklist

Scale the plan to the available time. State assumptions and do not guarantee a pass.
```

## Usage Guidelines

### How to Use These Prompts

1. Replace placeholders such as `[SERVICE_NAME]`, `[DIRECTORY_OR_REPOSITORY]`, and `[START_DATE]` with specific values.
2. Check the current official exam guide and code before generating or updating materials.
3. Adapt the scope to the repository's folder and naming conventions.
4. Combine prompts when useful, such as service notes followed by a gap review.
5. Request sources for version-sensitive claims and verify them before publishing.
6. Keep practice questions original and use them to learn the reasoning behind architecture choices.

### Best Practices

- State the business and technical requirements that drive each architecture decision.
- Ask for trade-offs among security, reliability, performance, cost, and operational complexity.
- Prefer scenario-based explanations over lists of service features.
- Separate confirmed AWS behavior from assumptions and recommendations.
- Use official AWS documentation, the current exam guide, and current pricing sources.
- Avoid unsupported coverage percentages, claims about actual exam content, and pass guarantees.
