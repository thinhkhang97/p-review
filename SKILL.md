---
name: p-review
description: Use this skill when the user asks about "performance review", "P-Review", "annual review", "year-end review", "self-assessment", "đánh giá hiệu suất", or wants to summarize their work accomplishments for a review period. This skill generates a structured performance review by analyzing GitHub contributions.
---

# Performance Review Generator (P-Review)

## When to Activate

- User asks for help with their performance review or P-Review
- User wants to summarize their yearly work accomplishments
- User mentions annual review, self-assessment, or year-end review
- User asks "what did I do this year" in a review context

## Overview

This skill generates a structured P-Review by:
1. Gathering the user's GitHub contributions (PRs, commits) across organizations
2. Categorizing work by project and impact
3. Asking for business context and metrics
4. Producing a performance review document tailored to the target audience

## Workflow

### Step 1: Gather Information

**IMPORTANT: Ask ONE question at a time.** Do not combine multiple questions into a single AskUserQuestion call. Wait for the user's answer before asking the next question. This keeps the conversation natural and avoids overwhelming the user.

**Question flow (in this exact order):**

1. **First**, auto-detect what you can silently (no question needed):
   - GitHub username: `gh api user --jq '.login'`
   - GitHub orgs: `gh api user/orgs --jq '.[].login'`
   - Git email: `git config user.email`

2. **Ask**: "Who will read this review?" (AskUserQuestion with options)
   - **Non-technical Manager** (e.g., CEO, COO, Product Manager) — Focus on business impact, revenue, user growth, strategic value. Minimize technical details.
   - **Tech Lead** (e.g., Engineering Manager, CTO, Senior Engineer) — Balance business impact with technical depth. Include architecture decisions, tech stack choices, code quality improvements.
   - **HR** — Focus on role responsibilities, growth, collaboration, and measurable outcomes. Keep language accessible.

3. **Ask**: "What year is this review for, and what is your name, department, and manager's name?" (single AskUserQuestion, free-text via "Other")

4. **Confirm**: Show the detected GitHub username and orgs, ask "Are these correct? Any orgs to add or remove?" (single AskUserQuestion with Yes/No + Other for corrections)

Then proceed to Step 2.

### Step 2: Discover Contributions (Run in Parallel)

For EACH organization, launch a **parallel Agent** (subagent_type: general-purpose) to gather data. Each agent should run:

```bash
# Search PRs
gh search prs --author=<username> --owner=<org> --created="<year>-01-01..<year>-12-31" --limit 100 --json title,repository,createdAt,state,url

# Search commits
gh search commits --author=<username> --owner=<org> --committer-date="<year>-01-01..<year>-12-31" --limit 100 --json repository,commit --jq '.[] | "\(.repository.fullName) | \(.commit.committer.date | split("T")[0]) | \(.commit.message | split("\n")[0])"'
```

**Important**: Also check for repos where the user might commit under a different identity:
```bash
git config user.email
```
Then search with email if the username search returns fewer results than expected.

### Step 3: Categorize by Project

After gathering all data:

1. **Group** PRs and commits by repository
2. **Identify themes** per project:
   - New features / product development
   - Bug fixes / stability improvements
   - Infrastructure / DevOps / CI/CD
   - AI / ML related work
   - Frontend / Backend / Mobile
3. **Rank** projects — prioritization depends on the target audience (see Prioritization by Audience below)
4. **Select top 5** projects for the review (merge related repos into one project if they belong to the same product)

### Step 4: Ask for Business Context

**IMPORTANT: Ask ONE question at a time.** Present the list of discovered projects first, then ask contextual questions sequentially.

**Question flow (in this exact order):**

1. **Show** the user a summary of discovered projects (grouped by product), then **ask**: "Do these project groupings and product names look correct? Please rename or merge any that should be different." (free-text via "Other")

2. **Ask**: "Do you have any achievements NOT visible in GitHub? (e.g., mentoring, process improvements, presentations, on-call rotations)" (free-text via "Other", with a "Nothing to add" option)

3. **Based on audience, ask ONE more targeted question:**

   - **If Non-technical Manager**: "For each product, do you have any business metrics? (e.g., user counts, revenue impact, cost savings, growth rates)"
   - **If Tech Lead**: "For each product, do you have any technical metrics? (e.g., test coverage, uptime, performance improvements, incidents prevented)"
   - **If HR**: "Can you share examples of cross-team collaboration or professional development activities this year?"

Then proceed to Step 5.

### Step 5: Generate the P-Review

Output the review in the following format. **All output must be in English.**

---

#### Output Format

```
## P-Review Year <YEAR> — <Employee Name>

### Q1. Key Tasks and Accomplishments

| No | Product | Project | Key Task | Key Accomplishment | Priority |
|----|---------|---------|----------|--------------------|----------|
| 1  | ...     | ...     | ...      | ...                | 1        |
| 2  | ...     | ...     | ...      | ...                | 2        |
| 3  | ...     | ...     | ...      | ...                | 3        |
| 4  | ...     | ...     | ...      | ...                | 4        |
| 5  | ...     | ...     | ...      | ...                | 5        |

### Q2. Main Accomplishments of the Year

**Answers:**

**1. <Accomplishment title>**
<2-3 sentence narrative>

**2. <Accomplishment title>**
<2-3 sentence narrative>

**3. <Accomplishment title>**
<2-3 sentence narrative>

**4. <Accomplishment title>**
<2-3 sentence narrative>
```

---

## Prioritization by Audience

The order of projects in Q1 and accomplishments in Q2 MUST reflect what the target audience values most.

### Non-technical Manager — Priority Order:
1. Products/features that drove user growth, revenue, or market expansion
2. Work that reduced costs or improved operational efficiency
3. Products launched from zero to production (new business capabilities)
4. Infrastructure that enabled faster delivery or business continuity
5. Internal tools that improved team productivity

### Tech Lead — Priority Order:
1. Complex technical challenges solved (architecture, scalability, performance)
2. Systems built from scratch demonstrating strong engineering decisions
3. Code quality improvements (testing, refactoring, technical debt reduction)
4. DevOps/infrastructure that improved developer experience or reliability
5. Technical mentoring, code reviews, or knowledge sharing

### HR — Priority Order:
1. Projects with the broadest organizational impact
2. Cross-functional collaboration and teamwork examples
3. Professional growth demonstrated through new responsibilities
4. Process improvements that benefited the team
5. Initiative and self-driven contributions

---

## Writing Guidelines

### Tone by Audience

#### If audience is Non-technical Manager:
- **Tone**: Professional, confident, impact-focused
- **Technical depth**: Minimal — translate everything into business outcomes
- **Key Task**: Describe WHAT was built/delivered, not HOW
- **Key Accomplishment**: Lead with business metrics (users, revenue, cost savings, time-to-market)
- **Q2 Narrative**: Connect work to company strategy and growth goals
- **Avoid**: Framework names, library names, architecture patterns, PR/commit counts

#### If audience is Tech Lead:
- **Tone**: Professional, technically informed, impact-aware
- **Technical depth**: Moderate — include architecture decisions, tech choices, and engineering quality
- **Key Task**: Include both what was built AND key technical decisions (e.g., tech stack, patterns used, testing approach)
- **Key Accomplishment**: Mix of business impact AND technical achievements (e.g., user metrics + test coverage + uptime)
- **Q2 Narrative**: Highlight technical leadership, code quality, architecture decisions, and mentoring
- **Include**: Tech stack, testing strategy, performance improvements, technical debt reduction

#### If audience is HR:
- **Tone**: Professional, accessible, growth-oriented
- **Technical depth**: Minimal — focus on role, responsibilities, and outcomes
- **Key Task**: Describe responsibilities and scope in plain language
- **Key Accomplishment**: Focus on deliverables, team collaboration, and measurable results
- **Q2 Narrative**: Emphasize professional growth, cross-team collaboration, and initiative
- **Avoid**: All technical jargon

### Common Guidelines (All Audiences)
- **Language**: Always English
- Do NOT use raw PR counts or commit counts as accomplishments
- Frame in terms of outcomes, not activities
- If no metrics are available, describe the strategic value (e.g., "accelerated time-to-market", "enabled the team to operate independently")
- 3-4 accomplishments in Q2, ordered by impact relative to the audience
- Each Q2 entry should tell a mini-story: challenge → action → result
- Include specific numbers wherever possible
- Avoid vague statements without supporting evidence
- Avoid overly modest or overly inflated language

## Examples

### Example for Non-technical Manager audience:

| No | Product | Project | Key Task | Key Accomplishment | Priority |
|----|---------|---------|----------|--------------------|----------|
| 1 | **Product A** | **Full-stack Development** | Designed and built an AI-powered financial advisor from scratch — both backend and user-facing web app. Features include budget management, goal tracking, and AI conversations. | Since launch, acquired **8,000+ users** (3,800+ authenticated) handling **1,000+ conversations/month**. Built from zero to production in under 6 months. | 1 |
| 2 | **Product B** | **CI/CD Migration** | Migrated build pipelines from a paid CI service to GitHub Actions for mobile app releases. | **Saves $100+/month** in build costs while improving deployment reliability and speed. | 2 |

### Example for Tech Lead audience:

| No | Product | Project | Key Task | Key Accomplishment | Priority |
|----|---------|---------|----------|--------------------|----------|
| 1 | **Product A** | **Backend Architecture** | Designed backend using Clean Architecture (NestJS): domain entities, use cases, repository pattern. Implemented real-time AI streaming via SSE, Redis caching, Supabase auth with RLS policies. Wrote comprehensive E2E tests. | Production system serving 8,000+ users with **zero critical incidents**. Achieved **80%+ test coverage**. Clean separation of concerns enables easy feature additions. | 1 |
| 2 | **Product A** | **Frontend & AI UX** | Built React frontend with real-time streaming chat, speech-to-text (Web Speech API), image upload with preview, and guided multi-step onboarding. Implemented SWR for data fetching with optimistic updates. | Responsive across web and mobile webview. Average session duration of 5+ minutes indicates strong user engagement. | 2 |
