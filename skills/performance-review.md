---
inclusion: manual
---

# Skill: Performance Review — Core Values Mapping

Generate a structured performance review accomplishment record that maps technical initiatives to organizational core values.

## Core Values Reference (Customize to Your Organization)

| Belief | Definition |
|--------|-----------|
| **Unwavering integrity** | We hold ourselves to the highest standards in every interaction we have with members, partners, and each other. |
| **A passion for service excellence** | We deliver unparalleled experiences, built on a legacy of caring, that exceed the emerging needs of members, communities, and one another. |
| **Personal and mutual accountability** | We commit to delivering on the promises we make as individuals and as members of one team, to drive exceptional personal and shared outcomes. |
| **Thinking big and moving fast** | We set bold, ambitious goals and pursue them with speed, intentionality and agility, harnessing innovation to deliver on our strategy. |
| **The power of inclusion** | We embrace our unique identities, experiences and points of view to advance our company and reflect our communities and members. |
| **Investing in ourselves** | We continuously learn and grow, with an eye to what's needed in the future, to address ever-changing member needs. |

## Procedure

1. **Gather initiative details** — ask the user for:
   - Initiative or project name
   - Objective (problem, risk, or business need addressed)
   - Their specific role and actions completed
   - Technology or systems involved
   - Verified scope (numbers, environments, boundaries)
   - Challenges encountered and how they were addressed
   - Collaboration (teams involved and how they worked together)
   - Verified results (measurable outcomes, evidence)
   - Business impact areas (reliability, security, resiliency, efficiency, supportability, risk reduction, compliance, customer experience, cost avoidance)
   - Any incidents or SLA data

2. **Select applicable Core Beliefs** — choose 2-4 beliefs that genuinely align with the work. Do not force-fit all six. Use these mapping guidelines:

   | Belief | Map when the person... |
   |--------|----------------------|
   | Unwavering integrity | Maintained transparency, followed proper change controls, communicated risks honestly, upheld standards even when shortcuts were available |
   | A passion for service excellence | Improved reliability/availability for customers or internal consumers, exceeded expectations, protected customer-facing services |
   | Personal and mutual accountability | Took ownership of delivery, followed through on commitments, drove results as both an individual and team member, completed defined scope |
   | Thinking big and moving fast | Introduced automation, modernized a platform, applied innovative approaches, moved with urgency and intentionality, scaled a solution |
   | The power of inclusion | Incorporated diverse perspectives, mentored others, shared knowledge across teams, made work accessible to broader audiences |
   | Investing in ourselves | Learned a new technology, developed new skills, prepared the environment for future needs, grew capability for the team |

   > **Note:** Replace the beliefs above with your organization's values. The mapping pattern works for any set of corporate values — match initiative actions to value definitions.

3. **Generate the review record** using this structure:

```markdown
# CSAA Initiative and Core Beliefs Review Record

## Header

| Field | Value |
|-------|-------|
| **Employee** | [Name] |
| **Job title** | [Title] |
| **Review period** | [Period] |
| **Manager** | [Manager] |
| **Date updated** | [Date] |

---

## Initiative Information

| Field | Value |
|-------|-------|
| **Initiative name** | [Name] |
| **Time frame** | [Period] |
| **Status** | [Completed / In Progress / Milestone Completed] |
| **Technology or service area** | [Area] |

---

## Objective

**What problem, risk, or business need did this initiative address?**

[Statement connecting the work to organizational need]

---

## My Role and Contribution

**What was I responsible for?**

[Role description — one sentence]

**Actions I personally completed:**

- [Specific action with strong verb]
- [Specific action with strong verb]
- [...]

---

## Scope

**Verified scope of the initiative:**

- Systems, applications, or devices: [Verified numbers]
- Environments or locations: [Verified scope]
- Users or services affected: [If verified]
- Other measurable scope: [If applicable]

---

## Challenges and Risk Management

**Key challenges, dependencies, or constraints:**

[Challenges]

**How I addressed them:**

[Specific responses — not just "overcame" but how]

---

## Collaboration

**Teams or stakeholder groups involved:**

[Teams]

**How I contributed to the shared outcome:**

[Specific collaboration actions]

---

## Verified Results

- [Result with number or verifiable fact]
- [Result]
- [Result]

**Evidence or reference:**

[Change records, dashboards, reports, tickets]

---

## Business Impact

[Checkboxes for applicable impact areas]

**Impact statement:**

[2-3 sentences translating technical result into organizational value]

---

## Performance Review Accomplishment

### Detailed Version

> [Action verb] [initiative and verified scope] by [key actions or approach], resulting in [verified result]. This work [business relevance] and demonstrated [applicable Core Beliefs].

### Concise Review-Form Version

> [2-3 sentence summary: action, scope, result, impact]

### One-Line Highlight

> [Single sentence starting with a strong action verb]

---

## CSAA Core Belief Alignment

[Only beliefs that genuinely apply — 2-4 entries]

### [Value Name]

[1-2 sentences explaining how the work demonstrated this value. Use the official value language as anchor.]

---

## Lessons Learned and Follow-Up

**What worked well?**
[Lessons]

**What would I improve next time?**
[Improvement]

**Follow-up actions or opportunities:**
- [Follow-up]
```

4. **Writing rules:**
   - Start every accomplishment with a strong action verb (Led, Engineered, Automated, Implemented, Coordinated, Validated, etc.)
   - Never use "helped with" or "worked on" — find the precise verb
   - Only include verified numbers — if unverified, ask the user or mark as estimate
   - Expand all acronyms on first use
   - Do not invent metrics, outcomes, or claims not provided by the user
   - Use "without unplanned service interruption" only when confirmed
   - Translate technical work into business language for the impact statement
   - Keep belief alignment statements specific to the initiative — no generic filler
   - Replace "CSAA Core Beliefs" references with your organization's value names

5. **Quality checklist before delivering:**
   - [ ] Starts with strong action verb
   - [ ] Individual contribution is clear and specific
   - [ ] Only verified scope and results included
   - [ ] Business relevance explained (why it mattered)
   - [ ] Language understandable beyond the immediate technical team
   - [ ] Core beliefs naturally fit — not forced
   - [ ] Acronyms expanded on first use
   - [ ] No unsupported claims
   - [ ] No overstatement of individual credit
   - [ ] Concise enough for a review conversation

6. **Output format:** Save as `Review_<InitiativeName>_<Year>.md` in the workspace root.

## Reference File

Customize the template with your organization's values and terminology:
#[[file:CSAA_Initiative_Review_Template.md]]
